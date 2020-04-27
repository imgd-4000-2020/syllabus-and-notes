# Visuals

## Simple custom node: 
- Create a custom node in a material shader, perhaps in a third-person project since we'll be using that for our audio demos.
- Hookup to emissive property
- Sample code
```
float3(.5 + sin(ResolvedView.RealTime)*.5, GetSceneTextureUV(Parameters))
```

## Using Shader files 
1. Add "RenderCore" module to build script
2. Edit main.cpp file for project
3. Be sure to include macro at end!!!!!!!!

// Copyright 1998-2019 Epic Games, Inc. All Rights Reserved.

```glsl
#pragma once

#include "CoreMinimal.h"

class FShaderTesting : public IModuleInterface {

public:
  virtual void StartupModule() override;
  virtual void ShutdownModule() override;
};

// cpp

#include "ShaderTesting.h"
#include "Modules/ModuleManager.h"
#include "Misc/Paths.h"
#include "ShaderCore.h"

void FShaderTesting::StartupModule() {
  FString ShaderDirectory = FPaths::Combine(FPaths::ProjectDir(), TEXT("Shaders"));
  AddShaderSourceDirectoryMapping("/Project", ShaderDirectory);
}

void FShaderTesting::ShutdownModule() {
  ResetAllShaderSourceDirectoryMappings();
}

IMPLEMENT_PRIMARY_GAME_MODULE( FShaderTesting, ShaderTesting, "ShaderTesting" );
```

# Audio

0. Use third-person template
1. Add BoxTrigger to scene
2. Make blueprint for level, make sure BoxTrigger is selected
3. Add Collision>OnActorBeginOverlap node
4. Play Sound at Location

## Sound cues
  - delays, modulations, mixing   

  - Add Audio Volume surrounding trigger box
    - Add Reverb effect 
    - Make sure to override attenuation in audio cue
    - Make sure to set min / max distance values correctly
  
