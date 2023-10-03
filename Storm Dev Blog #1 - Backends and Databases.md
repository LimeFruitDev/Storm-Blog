<div align="center">
    <img src="https://i.imgur.com/MmDn1Kn.png" alt="Storm Dev Blog #1">
    <h1>Storm Dev Blog #1</h1>
    <h2>Backends and Databases</h2>
</div>

---

Hey everyone,

I figured that since we're working on this in the background and it may take a while until we're able to let you guys help us test our s&box adventures, it might be worthwhile to start writing these short blogs showing our progress. These will be pretty technical in nature, but I'll try to cover the whole subject as much as possible.

This one will focus on the backend that we've recently completed for Storm - the new roleplay framework we are creating for s&box. Storm itself is a bit far from showing off by itself, but the backend is actually so simple that it's already in a working state.

### ␥ Why a Backend?

In order to allow the framework to keep **persistent** data even after the server shuts down (such as player data, characters, items, etc), we use something known as a Database. This is actually used in Garry's Mod as well - [Clockwork](https://github.com/CloudSixteen/Clockwork), [NutScript](https://github.com/rebel1324/NutScript), [Helix](https://github.com/NebulousCloud/helix), and [even DarkRP](https://github.com/FPtje/DarkRP) all use it. It's what allows communities like [SuperiorServers to have leaderboards](https://superiorservers.co/darkrp/leaderboard/money/), communities like [PERPHeads to have public ban lists](https://bans.perpheads.com/), and so much more. Pretty much all data that needs to be accessed often, be kept securely, and be comfortably interacted with, is saved on a database.

Those of you who have run Garry's Mod servers in the past will probably be able to quickly pick up on the fact, however, that none of this ever actually required another standalone backend application. The way it works in Garry's Mod is super simple, actually - you set up a database, either locally or on another server, and simply use a library like [MySQLOO](https://github.com/FredyH/MySQLOO) to connect to it.

Well, as it happens - this is not actually possible in s&box. There are multiple reasons for this, the most obvious of which is the [code access list](https://wiki.facepunch.com/sbox/AccessList), which restricts you from running non-whitelisted external (or system) code, primarily in order to protect player safety. You can obviously override this exclusively on the server, but then you'd need separated server and client assemblies, which are not currently supported by default nor are easily implemented anyway.

To make matters worse, unlike Garry's Mod, s&box does not come with a built-in SQLite database, and in order to save persistent data, you'll require an external database at practically any given time. There is no support for local development with a database out of the box.

<div align="center">
    <img src="https://i.imgur.com/i4g4Zvs.png" alt="Server Client Assembly">
</div>

This seems like a massive price to pay for player safety when Facepunch could really just allow separate server-side code, the same as they did in Garry's Mod. Well, the truth is, they have plans to implement that, but along the way, they revealed just how powerful these kinds of backends actually are.

These backends actually give you complete control over the data ever so quickly. You can use anything from MongoDB to PostgreSQL to a completely different solution, and your s&box server would be left almost entirely unchanged as long as the backend can handle the same incoming data. We can implement advanced caching solutions (and even Redis) with practically zero effort while maintaining a more secure and resilient backend - a server crash wouldn't even affect it at all.

In fact, using a backend like this, you can have multiple servers running simultaneously, writing to the same database, and it would behave perfectly fine with no additional effort.

### ␥ RPCs

I think many people who have been following s&box development are familiar with the term "Remote Procedure Call" or RPC. In s&box, it refers to the communication method between the client and the server, using ClientRPCs and, in the future, ServerRPCs. However, this isn't necessarily the case with Storm, which also uses RPCs to describe the messages sent from the server to the backend. These RPCs are defined in C# and are all derived from a class called `BaseRPC`, which looks like this (stripped down):

```csharp
public class BaseRpc
{
    [JsonPropertyName("uniqueId")] public ulong  UniqueId    { get; set; }
    [JsonPropertyName("type")]     public string MessageType { get; set; }
}
```

The backend uses these two properties to describe what action to take and how to reply to the message. The `UniqueId` field describes an unsigned 64-bit integer assigned to each message incrementally in order to allow callbacks to be called, and the `MessageType` field names the actual function which will be called on the backend, such as `commitPlayer`.

If we put the whole process into a graph, it'd look like this - with most of the details stripped away for simplicity:

<div align="center">
    <img src="https://i.imgur.com/MgPv3Jl.png" alt="RPC Process Graph">
</div>

Some transient processes weren't placed in that graph, such as assigning a unique identifier, but these are so simple they're not worthy of their own paragraphs. The most important thing to note here is that we've actually gone with WebSockets as opposed to a REST API, and the main reasons for this choice were actually simplicity (REST APIs are slightly more complex to code and maintain), security (no need for API keys or anything like that) and the fact that the backend and the server will both actually be able to tell when the other side has disconnected.

What I meant by simplicity is that it's so simple to use these, in fact, that a C# RPC definition and a backend handler side-by-side look like this:

<div align="center">
    <img src="https://i.imgur.com/swNE8KA.png" alt="C# RPC Definition and Backend Handler">
</div>

## The Future

The backend is usable but hardly complete. In fact, many features, such as security (IP Whitelisting? Authentication Keys?), plugins, and so forth, are left completely unimplemented with no agreed course of action on how they will be made as of yet. We've got a long way to go with the backend to reach a state in which it is perfect for future server owners and developers alike.

With this in mind, the project is completely open-source (so is Storm, actually), and anyone who is able and willing to contribute to its development is welcome to do so.
