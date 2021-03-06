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

## High-Level Project Overview
The suite would consist of the following sub-projects:
* Simple Motion V2 library
* G-Code library and controller
* Basic stand-alone CNC controller console application (Reads G-Code files and executes them against a set of Simple Motion controllers using the batched command API)
* Client-server framework (Framework providing an authenticated client-server orchestration using Web Sockets - The clients could be service applications running on controller machines or users via a web browser)
* Client-server CNC controller useful for CNC farms (A headless agent running on a machine with direct communication to the motor controllers connected to a cloud based multi-tenant server and a web-based user client allowing G-Code files to be run against remote agents)

## Simple Motion V2 Library
A full-featured strongly-typed asynchronous library written based around the reference C library. Initially only the buffered functionality is required, could be initially implemented by [P/Invoking](https://en.wikipedia.org/wiki/Platform_Invocation_Services) the reference library produced by [Granite Devices](http://granitedevices.com/wiki/SimpleMotion_V2#Library).

## G-Code Library / Controller
A simple G-Code parsing library compatible based on the [Mach4](http://machmotion.com/cnc-info/g-code.html), [EMC](http://linuxcnc.org/docs/html/gcode.html) and [RepRap](http://reprap.org/wiki/G-code) flavours. (See [RS274/NGC](https://www.nist.gov/customcf/get_pdf.cfm?pub_id=823374)) Parsing would be split into two parts, a stateless parser which converts text lines into G-Code blocks and a stateful machine-agnostic controller which converts the G-Code words into an abstract form of high-level commands. (E.g. Move to this position at this feedrate)

A trajectory planner would need to be created which takes the high-level commands, applies the individual axis trajectory constraints (Acceleration, deceleration and maximum speed) and outputs the low-level movement commands which are to be replayed directly by the motor drivers. The trajectory planner would need to look ahead to the next command and calculate the maximum possible start and end speeds for that command (Based on the maximum speeds for each axis). Depending on the direction change between the two commands, the machine may not need to slow down to a complete stop before the next command and this can be decided at this point before the acceleration and deceleration are calculated.

[Block Diagram](https://github.com/yngndrw/SimpleMotionCNC.Design/blob/master/CNC%20Controller%20-%20Block%20Diagram.png)

[Trajectory Planning Flow Chart](https://github.com/yngndrw/SimpleMotionCNC.Design/blob/master/CNC%20Controller%20-%20Trajectory%20Planner%20Flow%20Chart.png)

## Stand-Alone CNC Controller
A simple console application which links the Simple Motion and G-Code libraries. Responsible for executing a single G-Code file (From command-line arguments) against the local machine (Connected via RS485). The batched-command API would be used to prevent synchronisation issues and timing complexities.

## Client-Server Framework
A web-socket-based framework which provides a secure (Intended to be publically accessible on the internet) multi-tenant service which orchestrates commands and responces between different clients. Clients may either be native applications or Javascript in-browser applications. No assumptions are to be made in the library regarding protocols or usage.

## Web-Based CNC Controller
A secure multi-tenant CNC controller which allows a user to control remote CNC machines via a web-interface. The CNC machines would be a headless agent much like the stand-alone controller, but rather than executing G-Code files from command-line arguments it would respond to remote commands. Users would be able to both control and monitor their CNC machines via a web-interface, allowing both the execution of full G-Code files and of ad-hoc commands. Feedback can be provided through a 3D rendering drawn on the user's interface and possibly also via a web-cam stream.
