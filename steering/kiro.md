---
inclusion: auto
---
Always use the xanther-xce MCP tools for codebase understanding before reading files directly.

- Call `xce_get_context` as your FIRST step when starting any task. It combines search, architecture context, and tracing into one call.
- Use `xce_architecture_context` before modifying any file to understand its role, dependencies, and what calls it.
- Use `xce_impact_analysis` before making changes that affect multiple files to understand downstream effects.
- Use `xce_search` to find code by meaning when you need to locate specific functionality.
- Use `xce_trace` to understand how a function connects to the broader architecture.

Prefer XCE context over grep, find, or reading files for understanding code structure. XCE provides architectural context that file reading alone cannot.
