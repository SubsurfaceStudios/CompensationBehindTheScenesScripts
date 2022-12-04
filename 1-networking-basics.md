# Episode Start
Welcome to Compensation Behind the Scenes, a video series detailing the internal workings of our game Compensation and other related Computer Science concepts.  
In this first episode, we're going to go through a basic explanation of networking, how it applies to multiplayer games, and how we specifically use it in Compensation.

# Part 1 - What is networking?
In the context of computers, networking is the process of using a channel (typically the internet) to transmit information between two computers.
When referring to multiplayer games, we usually mean the process of keeping certain pieces of data the same between two peers, also known as Synchronization.

Let's start with an example.

Let's say we have two people in two seperate rooms. In those rooms, we have 3 blocks. Those rooms and blocks are the exact same, but they are seperate.
![image](https://user-images.githubusercontent.com/82341739/205516781-9e28b314-8bbb-4b20-aefd-89dd93b02604.png)

Now, let's give these two people a challenge. They need to keep both of these rooms 100% synchronized, by making sure the positions of these blocks
are the same in both rooms.

Initially, all is well. The state of the blocks in the rooms are synchronized.

That is, until the person on the left moves the bottom block on top of the stack. 

![image](https://user-images.githubusercontent.com/82341739/205516856-7a8e1754-2065-41bd-8b2d-99ba12dbae8c.png)


All of a sudden, the rooms are out of sync. Therefore, Left needs to communicate this change to Right.

Let's say Left explains what they did to Right. From there, Right can understand those changes and apply them on their own. After that, things are synchronized again.

![image](https://user-images.githubusercontent.com/82341739/205516904-4c51b854-1096-4db6-a275-bb3c411e7554.png)

Of course, this is a gross oversimplification. In actuality we communicate over the internet using protocols like UDP or WebSockets, but the concept remains similar.
The person in charge of controlling an object (the 'Authority', if you will) makes a change, then communicates that change to all the other clients.
From there, the other clients apply the change, and life goes on.

# Part 1 addendum - Problems

When both players' rooms are not the same, they are in an unknown state usually called Desync. This means that the person who made the change,
does not (and cannot) know what other players' rooms look like.
When *another* change occurs during this period, to the same object, from a different player, we have what is called a Race Condition.
Both clients are fighting to prove which one of them is correct, when, in fact, neither is. This is the disadvantage of a peer-to-peer model like we use in Compensation.
In other games, you will see a centralized game server making the decisions, as opposed to clients. Server-based networking, however, is not in the scope of this episode.

# Part 2 - Persistence

This is all well and good, while both players are in the room. However, what happens when a player leaves? They are no longer sending updates as to
the position, rotation, and other information of the objects in their room, and therefore, people joining afterwards will have no idea what state the room is in.
This issue is solved by transferring Authority of objects (that is, the user sending information about them) before leaving. In the event a player leaves before
transferring Authority of an object (or in the case that Authority is fixed, such as the player character themselves), other clients will automatically destroy
or delete the object in question. This is how we get rid of player characters after a user leaves, as they could leave for any number of reasons. That includes crashing,
which means you won't have time to send a "deleted" event on the player character leaving, and so it would leave a "ghost" of the player behind.

# Final Notes - RPCs and Properties

What we've been referring to as "syncing" data is actually more aptly described as "a stream of changes".
There are other types of data transmission, and that's what we'll quickly touch on here.
An RPC (or Remote Procedure Call) is a way of calling a "function" on another client's device.
Here's a code example in C# with Photon, you don't need to understand the code as much as you do the concept.

```csharp
// Sends a message "message" over the network to all other clients.
public void SendMessage(string message) {
  // Call the "messageRecieved" method on the other clients.
  photonView.RPC(nameof(messageRecieved), RpcTarget.Others, message);
}

void messageRecieved(string message) {
  // Do something with the message.
}
```

As you can see, the message is transmitted along with the name of the function, to the targets defined by RpcTarget.Others, meaning all other players in the room.
From there, we can do something with the message, but that function is only called on OTHER clients, not ourself. After all, what's the point of sending a message to yourself?
  
There's one other way of synchronizing data between clients, and that is Room or Player Custom Properties.
Custom Properties of both varieties are essentially a table of data. You have a `string` key (a string of text, if you will), and a value associated with it.

![image](https://user-images.githubusercontent.com/82341739/205517818-dc16c2a1-f309-45f6-9463-bd00060a8fac.png)


This table is synchronized automatically between clients by Photon, and is extremely useful for things like room state, player names, and more.

This has been episode 1 of Compensation Behind the Scenes, thank you for watching.
Make sure to tune in for the next video tomorrow, where we'll discuss step-by-step how rooms are loaded in Compensation, from the moment you hit "join" to the moment you leave.
