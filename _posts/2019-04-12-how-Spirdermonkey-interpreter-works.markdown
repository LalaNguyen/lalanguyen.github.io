---
layout: post
title: How Spidermonkey's interpreter works
date: 2019-04-12 15:11:51 +0000
categories: [research,spidermonkey]

---

Today, we look at how Spidermonkey's interpreter interprets a script tag.

The browser's main function is located in nsBrowser.cpp. When the browser starts, it initializes the XPCOMGlue and the XULRunner, which helps developing basic desktop applications. To initialize them, the browser calls InitXPCOMGlue and do_main, respectively. The do_main is a swapper function for the XRE_main, located in the nsAppRunner.cpp. This XRE_main will initializes the runtime package environment on which firefox will run. It loads browser profile (e.g., single thread or multi-thread), startup cache, init XPCOM2, and a cycle collector. Peridiodically, the collector wakes up and free any suspecious XPCOM pointers that have been sitting too long in the memory

XPCOM2, located at xpcom/build/XPCOMInit.cpp:NS_InitXPCOM2, is a critical component. It initializes quite few important subsystems that we are interested in. 
- The JS engine, located in js/src/vm/Initialization.cpp, is initialized in this procedure, followed by the Garbage Collection, and the JIT Ion. 
- XPCOM2 also creates the gComponentManager that brings up the layout module (related to render engine, CSS and color).
- XPCOM2 initializes gService, Telemetry, and message loop

Once the initialization phase finishes, the XRE_main creates 4 worker threads and runs.

When a user navigates to an url, the function ProcessRequest in ScriptLoader.cpp is triggered. The function calls EvaluateScript, which performs actual compilation to bytecode and execute the bytecodes.

EvaluateScript receives a script loader's request aRequest. It must distinguish whether a script request is byte code or source script. If the request contains byte code, EvaluateScript decodes the byte code and executes it. For the first time visit user, aRequest contains source code of the script. For this reason, the script must be compiled and executed. 

A script's source code can be extracted from aRequest using GetScriptSource(JScontext, aRequest). This returns a nsAutoString type which can only be printed out using NS_LossyConvertUTF16toASCII. Recall the the HTML's script is encoded with UTF16.

A bytecode compiled script can be obtained through GetScript(). However, printout the bytecode is not straightforward because JSScript struct is not within the scope of dom/script/ScriptLoader.cpp. To make JSScript avaiable for the ScriptLoader.cpp, we add #include "vm/JSScript.h" into the cpp file and in addition, we update LOCAL_INCLUDES in dom/script/moz.build to include 'js/src/'. With this, we can have access to the JSScript struct members such as code() and codeEnd(). To print out the byte code, the following snippet code is used:

```cpp
dom/script/ScriptLoader.cpp:EvaluatingScript(){
  
  script = exec.GetScript();
  jsbytecode* pc = script->code();
  jsbytecode* end = script->codeEnd();
  while(pc<end){
    printf("0x%X ",*pc);
    pc++;
  }
  printf("\n");  
}
```