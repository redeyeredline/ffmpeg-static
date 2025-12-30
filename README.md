# FFmpeg Static Builds

Docker images for FFmpeg with various codec support, built for Cliparr.

## Images

- `ghcr.io/redeyeredline/ffmpeg-static:7.1.1-nvidia` - FFmpeg with NVIDIA NVENC support
- `ghcr.io/redeyeredline/ffmpeg-static:7.1.1-base` - FFmpeg base build (CPU + QuickSync)

## Building

The images are automatically built and pushed via GitHub Actions when Dockerfiles are updated.

To build manually:

```bash
# NVIDIA version
docker build -t ghcr.io/redeyeredline/ffmpeg-static:7.1.1-nvidia docker/nvidia/

# Base version
docker build -t ghcr.io/redeyeredline/ffmpeg-static:7.1.1-base docker/base/
```

## Usage

These images are used as base images for Cliparr builds. They include both `ffmpeg` and `ffprobe` binaries.

