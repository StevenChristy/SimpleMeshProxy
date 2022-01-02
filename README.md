# SimpleMeshProxy
A simple UE 4.26+ mesh proxy supporting static rendering path. Based on the open source RuntimeMeshComponent by Triaxis Games.

## Purpose
The purpose of the SimpleMeshProxy is to provide a base mesh proxy that can be used for components derived from UMeshComponent. In one of my projects I wanted to create procedural mesh but found that the built in Procedural Mesh Component was too inefficient (way too many data structure conversions) and it lacked a static render path. The Runtime Mesh Component had a lot of great features but I only needed the static rendering path. This mesh proxy can allow one to more easily create custom runtime generated geometry without the overhead of the built in procedural mesh component and without introducing a dependency on a third party alternative. This header-only solution is simple to implement and can be customized as needed.

## Features

* Static Rendering Path
* RayTracing (untested)
* Streamlined and Easy to Customize

## Example

This code example assumes you've created a custom component derived from UMeshComponent and are creating a new instance of FMySceneProxy in the CreateSceneProxy method of that custom component. When a new FMySceneProxy is constructed it copies the geometry data from mesh sections of the component. Your custom component would implement the MeshSections array and provide commands for populating the geometry stored in each section.

```cpp
struct FMyMeshSection
{
	uint32 MaterialIndex;
	bool Visible;
	TArray<FDynamicMeshVertex> VertexBuffer;
	TArray<uint32> IndexBuffer;
};

class FMySceneProxy : public FSimpleMeshSceneProxy
{
public:
	FMySceneProxy(UMyAwesomeMeshComponent* Component) :
		FSimpleMeshSceneProxy(Component)
	{
		int32 SectionCnt = Component->MeshSections.Num();
		Sections.AddZeroed(SectionCnt);
		ERHIFeatureLevel::Type FL = GetScene().GetFeatureLevel();
		bShouldRenderStatic = !IsMovable();
		FSimpleMeshSectionOptions Options;
		Options.bCastsShadow = true;
		Options.bIsMainPassRenderable = Component->bRenderInMainPass;
		Options.bShouldRenderStatic = bShouldRenderStatic;
		
		for (int i = 0; i < SectionCnt; i++)
		{
			Options.bIsVisible = MeshSection.Visible;
			Sections[i] = new FSimpleMeshSceneSection(MeshSection.VertexBuffer, MeshSection.IndexBuffer, 
				Component->GetMaterial(MeshSection.MaterialIndex), Options, FL);
		}
    
		SectionsUpdated();
	}
};
```

## TODO
The following tasks have not yet been done
* Test the RayTracing Render code (I'll get around to it)
* Test/Porting to UE5 (a few months away)
* Test versions other than than 4.26
* Add ability to change visibility of a section without re-creating the scene proxy (soon).
