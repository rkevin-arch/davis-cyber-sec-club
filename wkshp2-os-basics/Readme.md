## Workshop Agenda
* Review
* Process Intro
* Process details
* strace - demo
* H4X0R LKMs

Topics Covered: OS review, Syscalls, Process


I'll try to work from a intuitive base, rather than technical.  
Later we'll "overlay" the technical details on top of our understanding.  

## Review: A Simple Plan

Let's review OS basics, by looking at our restaurant example.
Let's take this simple layout for our fast-food restaurant.  

    +-----------------------+
    |                Dining |
    |                 Area  |
    |                       |
    |                       |
    +-----------------------+
    |               Service |
    |               Counter |
    |                       |
    |                       |
    +-----------------------+
    |               Kitchen |
    |                       |
    |                       |
    |                       |
    +-----------------------+

---

       +       ^
    +--|-------|------------+
    |  |       |     Dining |       1. Customer walks to Service counter and places order.
    |  |       |       Area |       2. Worker takes order, goes to kitchen, starts prep.
    |  |       |            |       3. Worker finishes and returns with order.
    |  |1      | 4          |       4. Customer receives order. Transaction finished.
    +--v-------+------------+
    |               Service |
    |               Counter |
    |                       |
    |                       |
    +----+------^-----------+
    |    |      |   Kitchen |
    |    |      |3          |
    |    |      |           |
    |    v 2    |           |
    +----+------+-----------+

---
## Complex, but not complicated

_This is a great, but why are we doing this??_   
Because, this maps to our OS model of syscalls. 

    +-----------------------+                +-----------------------+
    |             Userspace |                |                Dining |
    |                       |                |                  Area |
    |                       |                |                       |
    |                       |                |                       |
    +-----------------------+                +-----------------------+
    |                Kernel |                |               Service |
    |                       |     ======     |               Counter |
    |                       |                |                       |
    |                       |     ======     |                       |
    +-----------------------+                +-----------------------+
    |              Hardware |                |               Kitchen |
    |                       |                |  (Oven, Fridge, Mixer |
    |                       |                |   Storeroom)          |
    |                       |                |                       |
    +-----------------------+                +-----------------------+

In the previous example, Userspace made a request to the Kernel.  
The Kernel starts the work and returns.  
When it is finished, the response is returned to the caller in Userspace.  

But it's missing couple things...   
In an OS, the user brings input and asks the Kernel for resources...   
So, let's change our restaurant a little bit.   

__Now__, the customers can bring a recipe on how to make a meal.    
The Chef (Kernel) uses the recipe (Program) to assemble the meal (output).  
(We can even imagine the customer bringing some special ingredients of their own as well.) 

The Resources here are kitchen appliances: Stove, Oven, Grill, Fridge, Mixer, Storeroom.  
The request to make use of those appliances is a **System Call**.

## Demo: strace?
- look at how `strace ls` works
- look at how `strace ps` works

---
## Accounting: Keeping track of state.

Two Customers come with two recipes.   

    +-----------------------+                +-----------------------+
    |                       |                |                       | 
    |   Recipe: Pasta       |                |   Recipe: Burgers     |
    |   =============       |                |   ===============     | 
    +-----------------------+                +-----------------------+
    |                       |                |                       | 
    | Thread 1:             |                | Thread 1:             |
    | ---------             |                | ---------             |
    | Boil Water            |                | Add Meat, Spices      |
    | Add Pasta             |                | Mix                   |
    | Cook 10 minutes       |                | Mold Patties          |
    | Wash under cold water |                | Grill 5 minutes       |
    |                       |                | Add Tomato, Lettuce,  |
    | Thread 2:             |                |  Onions, Cheese       |
    | ---------             |                |  to Buns              | 
    | Cook Tomatoes         |                | Place Patty on Bun    | 
    |  Add Meat             |                |                       | 
    |  Add Spices           |                |                       | 
    | Cook 10 minutes       |                |                       |     
    |                       |                |                       | 
    | Add Sauce to Pasta    |                |                       | 
    +-----------------------+                +-----------------------+

We should probably have a way to keep track of this stuff.  
Perhaps a list of all outstanding orders, and each entry keeping the state of the dish.  
That way we know what all is going on, and where each item is.  

The Kernel does this exact thing.  
* **Process list** keeps track of all the processes.  
* Each entry in the list is called a **Process Control Block**.

As we'll see, this allows us to be more efficient.  
If we can keep track of how the cooking is done, we can set more things in action.  

---
### Mutliprocessing: 2 Recipes, 1 Chef.

We only have 1 Stove and Grill! 

But, our chef is smart: 
* Doesn't watch pot to make it boil.  
* Doesn't need to stand idle when there's other work. 

Moving from one task to the other is a **Context Switch**.  
Much like the kitchen, moving from one task to another involves some overhead, same is true in OS.   
This typically invovles: State store, state instantiation, and state saving.  
(Note that "state" involves a lot more things in reality.)

So, effectively a process from a Kernel point of view is just an entry, and its associated state.  

---
### LKM to hide stuff: FiNAlLy some 1337 H4x0r moves!! 

Let's look at `strace ps` again and see what's going on.  

So to hide a process, is simple, just don't print the process from the list.

# References:
* Operating Systems: Three Easy Pieces - http://pages.cs.wisc.edu/~remzi/OSTEP/ (Great Free book, IMO)
* Operating System Concepts - aka "Dinosaur Book" - http://os-book.com/OS9/index.html

