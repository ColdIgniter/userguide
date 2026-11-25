# Design and Architectural Goals

Our goal for ColdIgniter is the capability and flexibility of CodeIgniter 4 in a small package, written in the CFML of ColdFusion MX 6.1.

To meet this goal we keep each component simple and reject anything that does not further the stated objective.

From a technical and architectural standpoint, ColdIgniter follows the objectives of CodeIgniter:

- **Dynamic Instantiation.** Components are loaded and routines executed only when requested, rather than globally. The system makes no assumptions about what may be needed beyond the minimal core resources. The events triggered by the HTTP request, and the controllers and views you design, determine what is invoked.
- **Loose Coupling.** Coupling is the degree to which components of a system rely on each other. The less components depend on each other, the more reusable and flexible the system becomes. The goal is a very loosely coupled system.
- **Component Singularity.** Singularity is the degree to which components have a narrowly focused purpose. In ColdIgniter, each component and its functions are highly autonomous, to allow maximum usefulness.

ColdIgniter is a dynamically instantiated, loosely coupled system with high component singularity. It strives for simplicity and flexibility in a small package.

!!! note "Difference from CodeIgniter"
    ColdIgniter is a port, so its design follows CodeIgniter 4 as closely as the CFML of ColdFusion MX 6.1 allows. Where the platform has no counterpart of a PHP feature, the code does the work in a plain way and says so in a comment at the top of the file. The file PARITY.md in the framework lists how much of CodeIgniter 4 is ported.
