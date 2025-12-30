# NitroInbox Binaries

Pre-built binaries for NitroInbox that aren't available in standard package managers.

## Current Binaries

### llama-finetune

The LoRA fine-tuning tool from [llama.cpp](https://github.com/ggerganov/llama.cpp). Used by NitroInbox to create personalized AI adapters based on user corrections.

## Building New Versions

### When to Build

- llama.cpp releases a new version with improvements/fixes
- Bug fixes needed for the fine-tuning tool
- Adding support for new platforms

### How to Build

1. **Go to Actions tab** → "Build llama-finetune" → "Run workflow"

2. **Fill in the parameters:**
   - `llama_cpp_ref`: The llama.cpp git ref to build from (tag, branch, or commit)
     - Use `master` for latest
     - Use a specific tag like `b4600` for reproducible builds
   - `release_version`: The version tag for this release (e.g., `v1.0.2`)
     - **Must be unique** - can't reuse existing version tags
     - Follow semver: `v{major}.{minor}.{patch}`

3. **Click "Run workflow"**

4. **Wait for build** (~3 minutes for Apple Silicon)

5. **Verify release** was created at: https://github.com/dotnetfactory/nitroinbox-binaries/releases

### Updating NitroInbox to Use New Version

After building a new version, update `app.config.ts` in NitroInbox:

```typescript
// app.config.ts
fineTuning: {
  binaryUrls: {
    'darwin-arm64':
      'https://github.com/dotnetfactory/nitroinbox-binaries/releases/download/v1.0.2/llama-finetune-macos-arm64.zip',
    // ... other platforms
  },
}
```

The app automatically detects version changes by comparing:
- Local version: stored in `~/Library/Application Support/NitroInbox/tools/llama-finetune.version`
- Config version: extracted from URL (`/download/v1.0.2/` → `v1.0.2`)

When versions differ, the old binary is deleted and the new one is downloaded.

## Platform Support

| Platform | Runner | Status |
|----------|--------|--------|
| macOS Apple Silicon | `macos-14` | ✅ Supported |
| macOS Intel | `macos-15` | 🔜 Can be added |
| Linux x64 | `ubuntu-22.04` | 🔜 Can be added |
| Linux ARM64 | `ubuntu-22.04-arm` | 🔜 Can be added |
| Windows | - | ❌ Not planned (use WSL) |

To add more platforms, update `.github/workflows/build-finetune.yml`.

## Build Configuration

The workflow builds with these CMake flags:

```bash
cmake .. \
  -DGGML_METAL=ON \           # Metal GPU acceleration (macOS)
  -DLLAMA_BUILD_EXAMPLES=ON \ # Build example tools
  -DLLAMA_BUILD_TOOLS=ON \    # Build CLI tools
  -DBUILD_SHARED_LIBS=OFF \   # Static linking (self-contained binary)
  -DLLAMA_STATIC=ON           # Static linking
```

Static linking ensures the binary works without requiring `libllama.dylib`.

## Troubleshooting

### Build fails with "No rule to make target"
- The llama.cpp ref might be too old or the API changed
- Try using `master` or a more recent tag

### Binary fails with "Library not loaded: libllama.dylib"
- Build wasn't static - ensure `BUILD_SHARED_LIBS=OFF` and `LLAMA_STATIC=ON`

### Version not detected by NitroInbox
- Version is extracted from URL pattern: `/download/v{version}/`
- Ensure release tag follows format: `v1.0.0`, `v1.0.1`, etc.
