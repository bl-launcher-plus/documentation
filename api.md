# api proposal

## objectives
* module communication (<->)
* dependency resolution
* error handling

----------

### module communication
#### structs, etc
```
struct blmodule {
	char[256] libraryName;
	void* init;
	void* deinit;
	DWORD imageBase;
	DWORD imageSize;
	bool initialized;
};

enum blerrors {
	 0: BLOADER_NO_ERROR,
	-1: BLOADER_NO_MODULE,
	-2: BLOADER_NO_INIT,
	-3: BLOADER_NO_DEINIT,
	-4: BLOADER_FAIL_INIT,
	-5: BLOADER_FAIL_DEINIT
};

```
#### proposed api
```int bloader_load(const char* module);```
* usermode, will be called by the user ingame or by add-ons
* will insert a blmodule struct into the global module table
* possible return values:
	* 0: loaded successfully
	* n < 0: did not load successfully, call bloader_geterror with the return value.

```const char* bloader_geterror(int errorcode)```
* get a string corresponding to the error code
* can be called from usermode or by the library internally.

```int bloader_unload(const char* module)```
* unload a module, calling it's deinitialization function.
* grab the module struct from the global module table
* also replace all of it's consolefunction declarations with a declaration that contains a function that does nothing-- avoiding the issue where unloading a dll and calling it's consolefunctions will result in a crash

* possible return values:
	* 0: unloaded successfully
	* n < 0: did not unload successfully, call bloader_geterror

```void* bloader_sym(blmodule* mod, const char* fnName)```
* **NOT A USERMODE FUNCTION**
* retrieve a pointer to an exported function from the module
* possible return values:
	* nullptr: could not find function
	* n != nullptr: a pointer to the exported function

### module communication
functions that will be available for modules to call

```
void bloader_consolefunction_string(const char* nameSpace, const char* name, StringCallback callBack, const char* usage, int minArgs, int maxArgs)
void bloader_consolefunction_bool(const char* nameSpace, const char* name, BoolCallback callBack, const char* usage, int minArgs, int maxArgs)
void bloader_consolefunction_int(const char* nameSpace, const char* name, IntCallback callBack, const char* usage, int minArgs, int maxArgs)
void bloader_consolefunction_void(const char* nameSpace, const char* name, VoidCallback callBack, const char* usage, int minArgs, int maxArgs)
void bloader_consolefunction_float(const char* nameSpace, const char* name, FloatCallback callBack, const char* usage, int minArgs, int maxArgs)
```
* will wrap ConsoleFunction, and call it with said parameters.
* will also store an entry inside of the module's function table, which we will refer to upon deinitialization of said module 

```
void bloader_consolevariable_int(const char* name, int* var)
void bloader_consolevariable_float(const char* name, float* var)
void bloader_consolevariable_string(const char* name, char* var)
void bloader_consolevariable_bool(const char* name, bool* var)
```
* expose a c++ variable to torquescript
* simple enough
```
int bloader_require(const char* module);
```
* will load a module into the global module table, and call it's initialization function




