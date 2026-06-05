Place your Unity WebGL build files here.

After building your Unity project for WebGL (File > Build Settings > WebGL > Build),
copy the contents of the Build folder from the output directory into this directory.

Expected file structure:
  public/unity-build/
  ├── Build/
  │   ├── Build.data
  │   ├── Build.framework.js
  │   ├── Build.loader.js
  │   ├── Build.wasm
  │   └── ...
  └── StreamingAssets/
      └── ...

If your build has a different name than "Build", update the paths in UnityViewer.vue props.
