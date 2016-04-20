# SimpleMotionCNC.Design 
*Design for CNC software which uses the [Simple Motion](http://granitedevices.com/wiki/SimpleMotion_V2) protocol for machine control.*

This high-level design covers the design decisions for a suite of CNC software intended for use with drives which speak the [Simple Motion](http://granitedevices.com/wiki/SimpleMotion_V2) protocol. While the design is technology agnostic, the intended technology choices are as follows:
* C# / [.Net Core](https://dotnet.github.io/):
  * Cross-platform runtime
  * Excellent base-class library
  * Large range of [available packages](https://www.nuget.org/)
  * [High performance runtime](https://blogs.msdn.microsoft.com/dotnet/2013/09/30/ryujit-the-next-generation-jit-compiler-for-net/)
  * Well-known syntax (C-based)
* Web Sockets for machine-to-machine communication
  * [Good Web Socket library support in .Net](http://signalr.net/) (.Net Core support in [SignalR 3 comming soon](https://github.com/aspnet/Home/wiki/Roadmap#future-work))
  * Low latency communication (Good for feedback or commands)
  * Bi-directional communication (Send commands, receive positional updates which can be plotted in 3D on the client)

*High-Level Project Overview*

The suite would consist of the following sub-projects:
* Simple Motion V2 library
* G-Code library
* Basic stand-alone CNC controller console application (Reads G-Code files and executes them against a set of Simple Motion controllers using the batched command API)
* Client-server framework (Framework providing an authenticated client-server orchestration using Web Sockets - The clients could be service applications running on controller machines or users via a web browser)
* Client-server CNC controller useful for CNC farms (A head-less agent running on a machine with direct communication to the motor controllers connected to a cloud based multi-tenant server and a web-based user client allowing G-Code files to be run against remote agents)
