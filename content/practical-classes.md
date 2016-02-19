Title: "Practical Classes"
Category: projects
Date: 2016-02-17 19:19:00

While there are many tutorials out there to impart the wisdom of classes through various incantations of "Cats are animals" there seems to be little in the way of more concrete "this is actually how I'd do it" examples.  This post attempts to fill that gap a bit.

Sometimes it seems that 50% of good code is just having good names for things.  There are various camps about how one should name variables and certainly every programming language has at least one strong opinion about captilization of class names.  In the end, though, it is the names that make sense to a human that are the most useful.  Let us start with a counter example.

## Books, Books, and more books

    :::C#
    namespace ClassDemoApp
    {
        class Book
        {
            public void open()
            {
                System.Diagnostics.Debug.WriteLine("Book is open!");
            }
        }

        class Book2 : Book
        {
            public void open()
            {
                System.Diagnostics.Debug.WriteLine("Book2 is open!");
            }
        }

        class Program
        {
            static void Main(string[] args)
            {
                Book2 book = new Book2();
                book.open();
            }
        }
    }

Okay so technically this is valid code but then so is this:

    :::C#
    namespace n
    {class b{public void open(){System.Diagnostics.Debug.WriteLine("b is open!");}}class b2:b{public void open(){System.Diagnostics.Debug.WriteLine("b2 is open!");}}

        class Program
        {
            static void Main(string[] args)
            {
                b2 b3 = new b2();
                b3.open();
            }
        }
    }

Okay but that is not terribly easy for a human to read and since it is the human who needs to get it right (and let's face it problems in software are problems in humans), it is beneficial to make it intelligible.

So let's revisit the above code with a better example:

    :::C#
    namespace ClassDemoApp
    {
        class Book
        {
            public string Author;
            public string ISBN;

            public void open()
            {
                System.Diagnostics.Debug.WriteLine("Book is open!");
            }
        }

        class NatrualLanguageDictionary : Book
        {
            public void open()
            {
                System.Diagnostics.Debug.WriteLine("Dictionary is open!");
            }

            public string LookUp(string word)
            {
                return String.Format("{0} is defined as, um, a word.", word);
            }
        }

        class Program
        {
            static void Main(string[] args)
            {
                // Here we use "userBook" to mean a specific instance belonging to the user
                Book userBook = new NatrualLanguageDictionary();
                userBook.open();
            }
        }
    }

This is a step in the right direction but I find there is a lot of power in not limiting my code to physical-world relationships.  OOP is, after all, a programming concept.  The above works and is fine but let's consider organizing those books:

    :::C#
    namespace ClassDemoApp
    {
        class Book
        {
            public string Author;
            public string ISBN;

            public void open()
            {
                System.Diagnostics.Debug.WriteLine("Book is open!");
            }
        }

        class NatrualLanguageDictionary : Book
        {
            public void open()
            {
                System.Diagnostics.Debug.WriteLine("Dictionary is open!");
            }

            public string LookUp(string word)
            {
                return String.Format("{0} is defined as, um, a word.", word);
            }
        }

        class Program
        {
            static void Main(string[] args)
            {
                IList<Book> userBooks = new List<Book>();
                userBooks.Add(new NatrualLanguageDictionary());
            }
        }
    }

We didn't do much here but we introduced a .NET class called `List`.  The reason I mention it is that it has *absolutely nothing* to do with Books.  The behaviors in these two classes are orthogonal and our class hierarchy can remain flat (and easily maintained).  At this point in my day-to-day coding I tend towards using interfaces rather than subclassing much of the time.  Subclassing is hugely powerful and easily abused.

Alright so let's get even more realistic using interfaces and a bit of subclassing.

    :::C#
    // Sad vector has no vector math.
    public struct Vector2
    {
        public float x;
        public float y;
    }

    // Components give Entities behavior
    public interface IComponent
    {
        void DoStuff();
        void SetParent(IEntity entity);
    }

    // All IEntities do things...
    public interface IEntity
    {
        Vector2 Position { get; set; }
        Vector2 Velocity { get; set; }

        void AddComponent(IComponent comp);
        void DoAllTheStuff();
    }

    // but Entities do them a specific way
    public class Entity : IEntity
    {
        IList<IComponent> components;

        public void AddComponent(IComponent comp)
        {
            components.Add(comp);
        }

        public void DoAllTheStuff()
        {
            foreach(var c in components)
            {
                c.DoStuff();
            }
        }
    }

    public class PhysicsComponent:IComponent
    {
        IEntity parent;

        public void DoStuff()
        {
            float time = 0.1f;
            parent.Position = new Vector2() { x = parent.Position.x + parent.Velocity.x * time, y = parent.Position.y + parent.Velocity.y * time };
        }

        public void SetParent(IEntity entity)
        {
            parent = entity;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            IEntity e = new Entity();

            e.AddComponent(new PhysicsComponent());

            e.DoAllTheStuff();
        }
    }

This sample is still not fully complete but it is closer to a real-world example.  We use subclassing to provide some default behavior in the base class (Entity) but consumers (such as Program.Main) don't actually require us to provide an Entity subclass.  Instead we only have to provide something that *looks* like an entity.  And the IEntity interface defines how they look - what methods and properties are guaranteed to exist.
