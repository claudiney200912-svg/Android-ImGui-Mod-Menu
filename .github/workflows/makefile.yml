#include <jni.h>
#include <pthread.h>
#include <unistd.h>
#include <android/log.h>

// Includes padrão de templates de mod menu
#include "Includes/imgui/imgui.h"
#include "Includes/Dobby/dobby.h"

#define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG, "LuaModding", __VA_ARGS__)

// Offsets fictícios (Você pega isso no Il2CppDumper - dump.cs)
// GorillaLocomotion.Player ou similar
#define OFFSET_PLAYER_UPDATE 0x6A4B20
#define OFFSET_GET_TRANSFORM 0x1A2B3C

// Globals
bool bFly = false;
bool bNoclip = false;
bool bSpeed = false;
void* localPlayer = nullptr;

// Structs básicas da Unity
struct Vector3 {
    float x, y, z;
};

// =================================================================================
// IMGUI MENU RENDER
// =================================================================================
void RenderMenu() {
    // Flags pra deixar o menu travado do jeito que modders gostam
    ImGui::Begin("lua modding", nullptr, ImGuiWindowFlags_NoCollapse | ImGuiWindowFlags_AlwaysAutoResize);

    // Créditos
    ImGui::TextColored(ImVec4(0.2f, 0.8f, 0.2f, 1.0f), "the mod is by s.t.l.o.x");
    ImGui::Separator();
    
    // Toggles
    ImGui::Checkbox("Fly", &bFly);
    ImGui::Checkbox("Noclip", &bNoclip);
    ImGui::Checkbox("Speed Boost", &bSpeed);

    ImGui::End();
}

// =================================================================================
// HOOKS
// =================================================================================
void (*old_PlayerUpdate)(void* instance);
void PlayerUpdate_Hook(void* instance) {
    localPlayer = instance;
    
    if (localPlayer != nullptr) {
        // Lógica injetada direto no update do player
        if (bFly) {
            // Normalmente manipula o Rigidbody velocity pra Vector3(0,0,0) e adiciona input das mãos
            // *(Vector3*)((uint64_t)localPlayer + 0x40) = Vector3{0, 0, 0}; 
        }
        
        if (bNoclip) {
            // Desativa mesh colliders das mãos
        }
        
        if (bSpeed) {
            // Altera maxJumpSpeed ou m_speed (exemplo de pointer math)
            *(float*)((uint64_t)localPlayer + 0x5C) = 15.0f; // Seta a velocidade pra 15
        } else {
            // Restaura o valor original se desativado (ex: 6.5f)
            // *(float*)((uint64_t)localPlayer + 0x5C) = 6.5f; 
        }
    }
    
    old_PlayerUpdate(instance); // Chama a original pra não crashar
}

// =================================================================================
// MAIN THREAD & INJECTION
// =================================================================================
void* HackThread(void*) {
    LOGD("lua modding injected!");
    
    // Espera o il2cpp.so carregar na memória (padrão em Unity)
    do {
        sleep(1);
    } while (dlopen("libil2cpp.so", RTLD_NOLOAD) == nullptr);

    LOGD("libil2cpp.so loaded. Applying hooks...");

    // Pega o endereço base do il2cpp na memória do aparelho
    // (Em um template real, você usaria uma função getAbsoluteAddress aqui)
    uint64_t il2cppBase = 0; // KittyMemory::getAbsoluteAddress("libil2cpp.so", 0);
    
    // Cria os hooks usando Dobby
    if (il2cppBase != 0) {
        DobbyHook((void*)(il2cppBase + OFFSET_PLAYER_UPDATE), (void*)PlayerUpdate_Hook, (void**)&old_PlayerUpdate);
        LOGD("Hooks applied successfully!");
    }

    return nullptr;
}

// JNI_OnLoad é a porta de entrada obrigatória para bibliotecas .so no Android
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) {
    JNIEnv* env;
    vm->GetEnv((void**)&env, JNI_VERSION_1_6);

    // Cria uma thread separada para não travar a UI principal do jogo
    pthread_t ptid;
    pthread_create(&ptid, nullptr, HackThread, nullptr);

    return JNI_VERSION_1_6;
}
