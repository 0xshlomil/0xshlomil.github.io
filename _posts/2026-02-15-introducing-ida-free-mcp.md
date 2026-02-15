---
layout: post
title: 'Introducing ida-free-mcp'
categories: [Reverse Engineering, IDA, MCP, AI]
---

How I used Claude to convert a Python IDA plugin to native C++ — because it sounded like a fun side project.

## Introduction

For reverse engineering, [ida-pro-mcp](https://github.com/mrexodia/ida-pro-mcp) is an excellent IDAPython plugin that exposes IDA's analysis capabilities to LLMs through MCP. I saw it and immediately wanted to try it out. The thing is, IDAPython requires a paid IDA Pro license, and IDA Free only ships with the C/C++ SDK — no Python support.

Now, sure, there's [Ghidra](https://ghidra-sre.org/) — it's free, open source, and has its own scripting ecosystem. But if you're used to IDA why not give it some MCP support. Sometimes you just want to stay in the ecosystem.

So this started as an interesting side project: could I take ida-pro-mcp's Python implementation and have Claude convert it to a native C++ plugin using IDA's free SDK? Turns out, yes.

## Getting Claude to Do the Heavy Lifting

Manually porting ~2000 lines of Python to C++ while navigating IDA's SDK documentation didn't sound like my idea of a good time. The ida-pro-mcp codebase implements the MCP protocol, registers tools that wrap IDA's Python API, and communicates over stdio. Converting this to C++ meant:

1. Reimplementing the MCP protocol layer in C++
2. Replacing all IDAPython API calls with their C/C++ SDK equivalents
3. Switching from stdio transport to an embedded HTTP server (native plugins run inside IDA's process, so stdio isn't an option)
4. Handling thread synchronization — because of course all IDA SDK calls must run on IDA's main thread

I fed Claude the Python source and the IDA SDK headers and let it go to work. The Python tool handlers mapped fairly directly to C++ SDK calls, though IDA's C API has its own quirks — output buffers instead of return values, `qstring` types everywhere, and the classic "this function exists but is documented nowhere" experience that every IDA plugin developer knows and loves.

The MCP protocol and JSON-RPC layer were written from scratch in C++ since those don't depend on IDA at all. Claude handled those without breaking a sweat.

The results? Honestly, they're good enough. All 32 tools work. The plugin is stable. I've been using it for actual reverse engineering work and haven't hit any showstoppers. Could a human C++ expert have written cleaner code? Probably. Do I care? Not really — it works, and it took a fraction of the time.

## Architecture

The plugin runs an HTTP server on `127.0.0.1:13337` inside IDA's process. Press **Ctrl+Alt+M** (or go to Edit > Plugins > MCP) and the server spins up:

```
IDA Plugin (plugin.cpp)
  └─ McpPlugmod
       ├─ McpProtocol (mcp.h)        ← Tool/resource registry, MCP methods
       │    └─ JsonRpcRegistry        ← JSON-RPC 2.0 dispatch
       └─ McpHttpServer (server.h)    ← HTTP routes, CORS, SSE
            └─ POST /mcp             ← Main MCP endpoint
```

The trickiest part was thread safety. The HTTP server runs on its own thread, but IDA really doesn't like it when you call SDK functions from anywhere other than the main thread. The plugin uses `execute_sync(MFF_WRITE)` to marshal tool calls back to the main thread, with timeout and cancellation support. Each tool handler is wrapped with an `ida_sync_tool()` macro so you don't have to think about it — which is exactly how threading should work.

All dependencies are vendored as header-only libraries — [nlohmann/json](https://github.com/nlohmann/json) for JSON, [cpp-httplib](https://github.com/yhirose/cpp-httplib) for the HTTP server, and [doctest](https://github.com/doctest/doctest) for tests. Zero external packages, zero package manager headaches.

## What You Get

32 tools across 6 groups:

**Core** — `int_convert`, `lookup_funcs`, `list_funcs`, `list_globals`, `imports`, `find_regex`

**Analysis** — `disasm`, `xrefs_to`, `callees`, `find_bytes`, `basic_blocks`, `find`, `export_funcs`, `callgraph`, `xrefs_to_field`

**Memory** — `get_bytes`, `get_string`, `get_int`, `get_global_value`, `patch`, `put_int`

**Modify** — `set_comments`, `rename`, `patch_asm`, `define_func`, `define_code`, `undefine`

**Types** — `declare_type`, `read_struct`, `search_structs`, `set_type`

**Stack** — `stack_frame`, `declare_stack`, `delete_stack`

Plus 11 MCP resources like `ida://idb/metadata` for binary metadata, `ida://idb/segments` for memory segments, `ida://cursor` for the current cursor position, and `ida://structs` for type library structures. Basically everything you'd want an AI to be able to poke at while helping you reverse a binary.

## Getting Started

Building requires CMake 3.16+, a C++17 compiler, and the IDA SDK:

```bash
export IDASDK=/path/to/idasdk
./build.sh
```

This builds the plugin, runs tests, and installs the `.so`/`.dll`/`.dylib` to your IDA plugins directory.

To connect from Claude Code:

```bash
claude mcp add ida-mcp -- http://127.0.0.1:13337/mcp
```

Or add to your `.mcp.json`:

```json
{
  "mcpServers": {
    "ida-mcp": {
      "url": "http://127.0.0.1:13337/mcp"
    }
  }
}
```

Once connected, you can ask Claude to disassemble functions, trace cross-references, search for byte patterns, rename variables, annotate code with comments — all directly within your IDA session. It's like having a reverse engineering assistant that never gets tired and never complains about your variable names.

## Summary

This was a fun side project to see how far Claude could take a Python-to-C++ conversion of a real-world IDA plugin. The answer: pretty far. If you're an IDA user who wants to try LLM-assisted reverse engineering without switching to a different tool, give it a shot.

The project is open source and available at [github.com/0xshlomil/ida-free-mcp](https://github.com/0xshlomil/ida-free-mcp).
