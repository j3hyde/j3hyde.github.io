Title: Why and how I made a WinRT Component
Category: articles
Date: 2017-01-08 11:58:00
Status: published

Now that Windows 10 is a thing and FMOD directly supports a module for it, this topic isn't likely to be important for many people but it was a "must have" for my game for Windows Phone 8.1.

# Impetus and Background (The Why)
I started a "simple" game project based on a puzzle concept that I built in JavaScript.  I had some extra motivation from the [2014](http://ludumdare.com/compo/2014/09/29/october-challenge-2014/) and [2015 October Challenge](http://ludumdare.com/compo/2015/09/28/october-challenge-2015/) along the way.  (No, I haven't finished yet but that's beside the point!)  To access a market (thereby also increasing complexity) I decided to "go mboile."  I wasn't planning to finish in that single month but again, the motivation was useful.  I chose Microsoft Labs' Win2D as teh graphics tech (Thank you, Shawn Hargreaves and team) and went with C# for implementing an engine.  With the core built I brought someone on board for audio and had a new requirement:  [FMOD](http://fmod.org "FMOD Site") support.

Now you might say "what's the big deal?  FMOD supports Windows Phone 8.1 and has C# bindings" and you'd be right excep that they don't have _C# bindings on Windows Phone 8.1_.  Thus I entered the world of Windows Run-Time Components and C++/CX.

The first line of research was to look into what my options were. It wasn't long before I put together just why FMOD lacked the module I needed.  THe easiest way to create C# bindings for a C/C++ library (DLL) is to use [PInvoke](https://msdn.microsoft.com/en-us/library/aa288468(v=vs.71).aspx).  If you look at most implementations of C# bindings that's what you'll find:  A thin wraper around a sugar-filled lollipop with a PInvoke center.

The next stop was to figure out how to avoid that "Binding Cavity" and introduce some API nutrition.  For better or worse Microsfot has Run-Time Components for just this purpose.  Once an RT component is built, it can be used from any language supported by the runtime.  So you coul djust as easily use that component in a JavaScript app as in a C++/CX or C# app (all on a Windows platform, of course).

# The Good Stuff (The How)
So, to the meat of it!  We have FMOD DLLs built for the target platforms - namely Windows Phone 8.1 ARM/x896 and Windows ARM/x86 (i.e. _normal Windows Desktop_) - and we have .libs and headers to use those DLLs in our C++/CX projects.  Let's take a look:

    :::C
    /*$ preserve start $*/

    /*
        fmod_studio.h - FMOD Studio API
        Copyright (c), Firelight Technologies Pty, Ltd. 2016.

        This header defines the C API. If you are programming in C++ use fmod_studio.hpp.
    */

    #ifndef FMOD_STUDIO_H
    #define FMOD_STUDIO_H

    #include "fmod_studio_common.h"

    #ifdef __cplusplus
    extern "C" 
    {
    #endif

    /*
        Global
    */
    FMOD_RESULT F_API FMOD_Studio_ParseID(const char *idString, FMOD_GUID *id);
    FMOD_RESULT F_API FMOD_Studio_System_Create(FMOD_STUDIO_SYSTEM **system, unsigned int headerVersion);

    /*$ preserve end $*/

    /*
        System
    */
    FMOD_BOOL   F_API FMOD_Studio_System_IsValid(FMOD_STUDIO_SYSTEM *system);
    FMOD_RESULT F_API FMOD_Studio_System_SetAdvancedSettings(FMOD_STUDIO_SYSTEM *system, FMOD_STUDIO_ADVANCEDSETTINGS *settings);
    FMOD_RESULT F_API FMOD_Studio_System_GetAdvancedSettings(FMOD_STUDIO_SYSTEM *system, FMOD_STUDIO_ADVANCEDSETTINGS *settings);
    FMOD_RESULT F_API FMOD_Studio_System_Initialize(FMOD_STUDIO_SYSTEM *system, int maxchannels, FMOD_STUDIO_INITFLAGS studioflags, FMOD_INITFLAGS flags, void *extradriverdata);
    FMOD_RESULT F_API FMOD_Studio_System_Release(FMOD_STUDIO_SYSTEM *system);
    FMOD_RESULT F_API FMOD_Studio_System_Update(FMOD_STUDIO_SYSTEM *system);

It's a pretty straight-forward C API here.  It was refreshing to see a commercial API like this.  It seemd to have a few bits of "history" but with a strong dose of intent.  Now for C++:

    :::C++
    class System
    {
    private:
        // Constructor made private so user cannot statically instance a System class. System::create must be used.
        System();
        System(const System &);

    public:
        static FMOD_RESULT F_API create(System **system, unsigned int headerVersion = FMOD_VERSION);
        FMOD_RESULT F_API setAdvancedSettings(FMOD_STUDIO_ADVANCEDSETTINGS *settings);
        FMOD_RESULT F_API getAdvancedSettings(FMOD_STUDIO_ADVANCEDSETTINGS *settings);
        FMOD_RESULT F_API initialize(int maxchannels, FMOD_STUDIO_INITFLAGS studioflags, FMOD_INITFLAGS flags, void *extradriverdata);
        FMOD_RESULT F_API release();

        // Handle validity
        bool F_API isValid() const;

The C++ API is a thin wrapper on the C API.  For my purposes I really only needed FMOD Studio bindings but I did build around a portion of the base FMOD API as well.

Using the C++ API as a direction, I proceeded to export and otherwise rebind in C++/CX.  The tricky thing is that you can only export Windows Run-Time types such as int, char, classes that reference only those types in their public interfaces, or classes referencing those classes.  Note you can still use non-RT types in non-public contexts.  One primary distinction is that you canot use pointers in the RT interfaces.  Instead CX provides reference types.  That means the RT Component had to manage memory for the C/C++ library while exposing references to the consuming application.

Okay, so what does the component really look like?  Here is System.h declaring the RT class that wraps the FMOD System:

    :::C++
    namespace FMODStudio
    {
        public ref class System sealed
        {
        internal:
            FMOD::Studio::System* internalSystem;
            // Constructor made private so user cannot statically instance a System class.  SystemFactory::create must be used.
            
            System(FMOD::Studio::System* system);
            
        public:
            static RESULT create(System^* system);
            RESULT setAdvancedSettings(ADVANCEDSETTINGS settings);
            RESULT getAdvancedSettings(ADVANCEDSETTINGS* settings);

            // Init/Close.
            RESULT initialize(int maxchannels, INITFLAGS studioflags, FMODComponent::INITFLAGS flags, uint32 extradriverdata);

            RESULT release();
            RESULT loadBankFile(Platform::String^ name, LOAD_BANK_FLAGS flags, Bank^* bank);
            RESULT flushCommands();
            RESULT update();
        };
    }

If you're familiar with FMOD Studio's API then you'll note that this is nowhere near the entire API.  I've only implemented what I really needed at the time but it does highlight the types of changes required:

* Using references rather than pointers.  Things like `Platform::String^ name` and `Bank^\* bank` are RT references.
* All RT classes must be `ref` `sealed`.
* All types/classes used in the public interface must be RT types/classes.

Practically speaking, in addition to the type changes above, this means the class must manage memory internally and can't return pointers to allocated memory.  Here's a snippet of System.cpp:


    :::C++
    namespace FMODStudio
    {
        System::System(FMOD::Studio::System* system)
        {
            internalSystem = system;
            std::cout << "system:" << system << "\n";
        }

        RESULT System::create(System^* system)
        {
            FMOD::Studio::System* fmodStudioSystem;

            RESULT result = (RESULT)FMOD::Studio::System::create(&fmodStudioSystem);
            Util::PrintPointer(__FILE__, __LINE__, &fmodStudioSystem);
            Util::PrintPointer(__FILE__, __LINE__, fmodStudioSystem);

            if (result != RESULT::OK)
            {
                fmodStudioSystem->release();

                return result;
            }
            *system = ref new System(fmodStudioSystem);
            return result;
        }
    }

Fully learning that and then really "getting it" was a process but basically I tied the FMOD object life-times to RT objects in order to prevent leaks and otherwise made all pointer usage internal to the component.

In the end I had a run-time component for FMOD that I could use to replace the simple/hacked playback component in my audio system.  Fortunately I'd already designed that as a pluggable system so it was quite easy to drop in an FMOD audio provider.  I can now instantiate FMOD effects, set their parameters and positions, and play them within my mobile game that is otherwise written in C#.

# Addendum: Python
For a later, quick, small, side-project over New Year, I wanted to use FMOD from Python.  Using ctypes on Windows this was much easier than I had anticipated.  See [This post](http://www.fmod.org/questions/question/how-to-use-fmod-from-a-python-script/) for more information.

From a practicality standpoint I did end up wrapping the API in some Python classes.

<script src="https://bitbucket.org/j3hyde/midiplayground/src/738d8db7ad4d2a835fffb9f5ac908c73e3bb6920/fmod.py?embed=t"></script>

Note how straight-forward that is though.  I really just wanted to abstract the ctypes calls a bit so I didn't have to peppar them throughout my project.  Later on I built a simple sound event system that would call into that FMOD layer.  The bonus of course is that I could replace FMOD if I needed to.  The events were named more for their content than for how they'd be played.
