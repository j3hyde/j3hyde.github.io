Title: "Practical Classes"
Category: articles
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

But that is not terribly easy for a human to read and since it is the human who needs to get it right (and let's face it problems in software are problems in humans), it is beneficial to make it intelligible.

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

This is a step in the right direction but I find there is a lot of power in not limiting my code to physical-world relationships.  OOP is, after all, a programming concept not a physical one.  The above works and is fine but let's consider organizing those books:

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

We didn't do much here but we introduced a .NET generic class called `List`.  The reason I mention it is that it has *absolutely nothing* to do with Books.  The behaviors in these two classes are orthogonal and our class hierarchy can remain flat (and easily maintained).  At this point in my day-to-day coding I tend towards using interfaces rather than subclassing much of the time.  Subclassing is hugely powerful and easily abused.

## Better Game Entities for Home and Abroad

Alright so let's get even more realistic using interfaces and a bit of subclassing.  First an extreme "anti-example":

    :::C#
    namespace ClassDemoApp
    {
        // Sad vector has no vector math.
        public struct Vector2
        {
            public float x;
            public float y;
        }

        // Entities do stuff
        public class Entity
        {
            Vector2 Position { get; set; }
            Vector2 Velocity { get; set; }

            public virtual void DoAllTheStuff()
            {
                // Stuff gets done in here
            }
        }

        // Lets have characters in our game
        public class Character : Entity
        {
            public virtual void DoAllTheStuff()
            {
                // Perhaps characters say things in the game
                this.SayStuff()
            }

            public virtual void SayStuff()
            {
                // Speak poetry to me...
            }
        }

        // Ah but we need AI characters
        public class NPC : Character
        {
            public virtual void DoAllTheStuff()
            {
                // Perhaps characters say things in the game
                this.SayStuff()

                // Also do AI movement things
            }

            public virtual void SayStuff()
            {
                // Speak poetry to me...
            }
        }

        // But this one is a player character
        public class Player : Character
        {
            public virtual void DoAllTheStuff()
            {
                // Perhaps characters say things in the game
                this.SayStuff()

                // Interpret player input here
            }

            public virtual void SayStuff()
            {
                // Say things worthy of a Player
            }
        }

        // Ah we want pickup items as well!
        public abstract class Item : Entity
        {
            public virtual void DoAllTheStuff()
            {
                // Detect interactions with the player
                self.DoPickup();
            }
            
            public virtual void DoPickup();
        }

        // ...like health items
        public class Health : Item
        {
            public virtual void DoPickup()
            {
                // add health to the player
            }
        }

        // ...and super health items
        public class SuperHealth : Health
        {
            public virtual void DoPickup()
            {
                // increase max health
            }
        }

        // ...and invincibility
        public class Invincibility : SuperHealth
        {
            public virtual void DoPickup()
            {
                // really go over the top here
            }
        }

        class Program
        {
            static void Main(string[] args)
            {
                Entity e = Player();
                SuperHealth h = new SuperHealth();
            }
        }
    }

Okay so that's a pathological case and is getting a bit silly really but I hope you can see how deep this tree is getting.  The problem is that as you add more kinds of items or characters that tree starts branching thereby compounding the problem.  What is the problem?  Maintainability, readability, flexibility.  It becomes extremely hard to reuse any of your code because you've made so many specific subclasses that when you go to subclass again, it's because you need just one method from the parent class but none of the other 20 methods that conflict with the new class you are trying to create.  Thus you make yet another subclass.  I'm sure better developers than I would name off a 200 item list of all the things wrong with this.

Let's abstract this and make the structure more flat.

    :::C#
    namespace ClassDemoApp
    {
        // Sad vector still has no vector math.
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
                comp.SetParent(this);
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
                var p = parent;
                if (p == null) return;

                p.Position = new Vector2() { x = p.Position.x + p.Velocity.x * time, y = p.Position.y + p.Velocity.y * time };
            }

            public void SetParent(IEntity entity)
            {
                parent = entity;
            }
        }

        public interface IPickupComponent : IComponent
        {
            public void DoPickup(IEntity player);
        }

        public class HealthPickupComponent: IPickupComponent
        {
            int health = 10;

            public void DoPickup(IEntity player)
            {
                // Apply the health
            }
        }

        class Program
        {
            static void Main(string[] args)
            {
                IEntity player = new Entity();
                player.AddComponent(new PhysicsComponent());
                player.DoAllTheStuff();

                IEntity healthPickup = new Entity();
                healthPickup.AddComponent(new HealthPickupComponent());
                healthPickup.DoPickup(player);
            }
        }
    }

This sample is still not fully complete but it is closer to a real-world example.  We use subclassing to provide some default behavior in the base class (Entity) but consumers (such as Program.Main) don't actually require us to provide an Entity subclass.  Instead we only have to provide something that *looks* like an entity.  And the IEntity interface defines how they look - what methods and properties are guaranteed to exist.

What you might start to see in this example is a different kind of complexity emerging.  Now I have to manage a bag of components that aren't guaranteed to exist at any given time and it's those components that have the properties and methods i need to interact with.  In my projects I end up with methods on the Entity class to help with finding contained components with certain interfaces.  For example, I want a physics service to be able to find the PhysicsComponent on each of two entities easily and quickly.  Another useful part of the Entity functionality is to define a way to clone Entities.  This allows me to define a prototypical "Monster" or "HealthPickup" Entity which has all the right components.  I can then clone that into a new Entity and spawn it in the game world.

What this comes down to is a sort of weighing the minimum cost involved.  The first might be easier to implement at first but the second method is more scalable.  In this example, you are moving from thinking of subclasses as different types of _things_ to thinking of them as different kinds of _behaviors_.  The concept of 'type' then becomes data on an object rather than a systemic property of the class itself.

Right.  So is that going to help anybody out there?  I don't know but I'd like to hear your thoughts.  Tweet me at @j3hyde.
