# Modify property

    Change "dll" to "lib" in property configuration type.
    Change "EASYHOOK_EXPORTS" to "EASYHOOK_LIB" in property preprocessor.

# Modify begining of easyhook.h

    #ifdef EASYHOOK_EXPORTS
        #define EASYHOOK_API                            __declspec(dllexport) __stdcall
        #define DRIVER_SHARED_API(type, decl)           EXTERN_C type EASYHOOK_API decl
    #else
        #ifdef EASYHOOK_LIB
            #define EASYHOOK_API                        __stdcall
            #define DRIVER_SHARED_API(type, decl)       typedef type EASYHOOK_API PROC_##decl; EXTERN_C type EASYHOOK_API decl
            
            EXTERN_C BOOL APIENTRY EasyHookDllMain(HMODULE hModule,
                DWORD  ul_reason_for_call,
                LPVOID lpReserved
            );
        #else
            #ifndef DRIVER
                #define EASYHOOK_API                    __declspec(dllimport) __stdcall
                #define DRIVER_SHARED_API(type, decl)	EXTERN_C type EASYHOOK_API decl
            #else
                #define EASYHOOK_API                    __stdcall
                #define DRIVER_SHARED_API(type, decl)	typedef type EASYHOOK_API PROC_##decl; EXTERN_C type EASYHOOK_API decl
            #endif
        #endif
    #endif

# Modify DllMain

    Change DllMain to EasyHookDllMain

# Use static lib

    #include "../include/easyhook/easyhook.h"

    #ifndef _M_X64
        #pragma comment( lib, "../include/easyhook/EasyHook32.lib")
        #pragma comment( lib, "../include/easyhook/AUX_ULIB32.lib")
    #else
        #pragma comment( lib, "../include/easyhook/EasyHook64.lib")
        #pragma comment( lib, "../include/easyhook/AUX_ULIB64.lib")
    #endif

    #pragma comment( lib, "psapi.lib")

    BOOL APIENTRY DllMain( HMODULE hModule,
                        DWORD  ul_reason_for_call,
                        LPVOID lpReserved
                        )
    {
        EasyHookDllMain((HMODULE)hModule, ul_reason_for_call, lpReserved);
        switch (ul_reason_for_call)
        {
        case DLL_PROCESS_ATTACH:
            // Disables the DLL_THREAD_ATTACH and DLL_THREAD_DETACH notifications for the specified dynamic-link library (DLL).
            // This can reduce the size of the working set for some applications.
            DisableThreadLibraryCalls((HINSTANCE)hModule);
        case DLL_THREAD_ATTACH:
        case DLL_THREAD_DETACH:
        case DLL_PROCESS_DETACH:
            break;
        }
        return TRUE;
    }

    // EasyHook will be looking for this export to support DLL injection. If not found then 
    // DLL injection will fail.
    extern "C" __declspec(dllexport) void __stdcall NativeInjectionEntryPoint(REMOTE_ENTRY_INFO* InRemoteInfo)
    {
        std::cout << "Injected by process Id: " << InRemoteInfo->HostPID << "\n";
        std::cout << "Passed in data size: " << InRemoteInfo->UserDataSize << "\n";
    }
