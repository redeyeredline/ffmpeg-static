# syntax=docker/dockerfile:1

# Build FFmpeg with comprehensive hardware acceleration support for Cliprr
# Includes: NVIDIA NVENC, Intel QSV, AMD AMF/VCE, VA-API, and CPU encoding
FROM ubuntu:22.04 AS builder

# Set environment variables
ENV NVIDIA_DRIVER_CAPABILITIES="compute,video,utility"
ENV NVIDIA_VISIBLE_DEVICES="all"
ENV DEBIAN_FRONTEND=noninteractive
ENV PATH="/usr/local/cuda/bin:${PATH}"
ENV LD_LIBRARY_PATH="/usr/local/cuda/lib64:${LD_LIBRARY_PATH}"

# Install build dependencies, CUDA, Intel Media SDK, and VA-API
RUN apt-get update && apt-get install -y \
    autoconf \
    automake \
    build-essential \
    cmake \
    git \
    libass-dev \
    libbz2-dev \
    libfontconfig-dev \
    libfreetype-dev \
    libfribidi-dev \
    libharfbuzz-dev \
    liblzma-dev \
    libmp3lame-dev \
    libnuma-dev \
    libogg-dev \
    libopus-dev \
    libspeex-dev \
    libtheora-dev \
    libtool \
    libtool-bin \
    libvorbis-dev \
    libx264-dev \
    libx265-dev \
    libxml2-dev \
    libvpx-dev \
    libva-dev \
    libdrm-dev \
    m4 \
    make \
    nasm \
    pkg-config \
    tar \
    zlib1g-dev \
    yasm \
    wget \
    curl \
    gnupg2 \
    software-properties-common \
    && wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.0-1_all.deb \
    && dpkg -i cuda-keyring_1.0-1_all.deb \
    && apt-get update \
    && apt-get install -y cuda-toolkit-12-1 \
    && git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git /tmp/nv-codec-headers \
    && cd /tmp/nv-codec-headers \
    && make install \
    && rm -rf /var/lib/apt/lists/* /tmp/nv-codec-headers

# Create build directory
WORKDIR /tmp/ffmpeg-build

# Download FFmpeg source
ARG FFMPEG_VERSION=7.1.1
RUN wget https://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.bz2 && \
    tar -xf ffmpeg-${FFMPEG_VERSION}.tar.bz2 && \
    cd ffmpeg-${FFMPEG_VERSION}

# Configure FFmpeg with comprehensive hardware acceleration support
# NVIDIA NVENC, Intel QSV (via VA-API), AMD VCE (via VA-API), and CPU encoding
# Note: libvpl and AMF require external SDKs not in Ubuntu repos, so we use VA-API for Intel/AMD
WORKDIR /tmp/ffmpeg-build/ffmpeg-${FFMPEG_VERSION}
RUN ./configure \
    --prefix=/usr/local \
    --enable-gpl \
    --enable-nonfree \
    --enable-nvenc \
    --enable-cuda-nvcc \
    --enable-libx264 \
    --enable-libx265 \
    --enable-libmp3lame \
    --enable-libopus \
    --enable-libvorbis \
    --enable-libass \
    --enable-libfreetype \
    --enable-libfribidi \
    --enable-libtheora \
    --enable-libvpx \
    --enable-libspeex \
    --enable-libxml2 \
    --enable-vaapi \
    --enable-postproc \
    --enable-pthreads \
    --enable-shared \
    --extra-cflags="-I/usr/local/cuda/include" \
    --extra-ldflags="-L/usr/local/cuda/lib64"

# Build FFmpeg (this builds ffmpeg, ffprobe, ffplay, etc.)
RUN make -j$(nproc)

# Verify ffprobe was built in the build directory
RUN ls -la /tmp/ffmpeg-build/ffmpeg-${FFMPEG_VERSION}/*probe* || echo "Checking for ffprobe in build dir..."
RUN test -f /tmp/ffmpeg-build/ffmpeg-${FFMPEG_VERSION}/ffprobe || (echo "ERROR: ffprobe not built!" && ls -la /tmp/ffmpeg-build/ffmpeg-${FFMPEG_VERSION}/ | grep -E "(ffmpeg|ffprobe)" && exit 1)

# Install FFmpeg (this installs all built binaries including ffprobe)
RUN make install

# List what was actually installed
RUN echo "=== Binaries in /usr/local/bin ===" && ls -la /usr/local/bin/ | grep -E "(ffmpeg|ffprobe)" || ls -la /usr/local/bin/

# Verify both ffmpeg and ffprobe were built and installed
RUN test -f /usr/local/bin/ffmpeg || (echo "ERROR: ffmpeg binary not found after make install" && exit 1)
RUN test -f /usr/local/bin/ffprobe || (echo "ERROR: ffprobe binary not found after make install" && exit 1)
RUN /usr/local/bin/ffmpeg -version | head -1
RUN /usr/local/bin/ffprobe -version | head -1

# Update library cache
RUN ldconfig

# Final stage - minimal runtime image
FROM ubuntu:22.04

# Install minimal runtime dependencies including VA-API and Intel Media drivers
RUN apt-get update && apt-get install -y \
    libass9 \
    libbz2-1.0 \
    libfontconfig1 \
    libfreetype6 \
    libfribidi0 \
    libharfbuzz0b \
    libjansson4 \
    liblzma5 \
    libmp3lame0 \
    libnuma1 \
    libogg0 \
    libopus0 \
    libsamplerate0 \
    libspeex1 \
    libtheora0 \
    libvorbis0a \
    libx264-163 \
    libvpx7 \
    libxml2 \
    libz1 \
    libx265-199 \
    libvorbisenc2 \
    libflac8 \
    libva2 \
    libva-drm2 \
    libva-x11-2 \
    libdrm2 \
    intel-media-va-driver-non-free \
    && rm -rf /var/lib/apt/lists/*

# Copy FFmpeg and FFprobe from builder (with explicit verification)
COPY --from=builder /usr/local/bin/ffmpeg /usr/local/bin/ffmpeg
COPY --from=builder /usr/local/bin/ffprobe /usr/local/bin/ffprobe
COPY --from=builder /usr/local/lib /usr/local/lib

# Verify files were copied successfully and list what we have
RUN ls -la /usr/local/bin/ | grep -E "(ffmpeg|ffprobe)" || (echo "ERROR: Missing binaries after COPY!" && ls -la /usr/local/bin/ && exit 1)
RUN test -f /usr/local/bin/ffmpeg || (echo "ERROR: ffmpeg not copied!" && exit 1)
RUN test -f /usr/local/bin/ffprobe || (echo "ERROR: ffprobe not copied!" && exit 1)

# Set up environment
ENV PATH="/usr/local/bin:${PATH}"
ENV LD_LIBRARY_PATH="/usr/local/lib:${LD_LIBRARY_PATH}"

# Update library cache
RUN ldconfig

# Test FFmpeg installation
RUN ffmpeg -version | head -1

# Test FFprobe installation
RUN ffprobe -version | head -1

# Verify both binaries exist in final image
RUN test -f /usr/local/bin/ffmpeg || (echo "ERROR: ffmpeg not found in final image" && exit 1)
RUN test -f /usr/local/bin/ffprobe || (echo "ERROR: ffprobe not found in final image" && exit 1)

# Test hardware acceleration support
RUN echo "=== Testing Hardware Acceleration Support ===" && \
    (ffmpeg -hide_banner -encoders | grep nvenc && echo "✓ NVENC available") || echo "✗ NVENC not available" && \
    (ffmpeg -hide_banner -encoders | grep vaapi && echo "✓ VA-API available (Intel QSV/AMD VCE)") || echo "✗ VA-API not available" && \
    (ffmpeg -hide_banner -encoders | grep libx264 && echo "✓ CPU H.264 (libx264) available") || echo "✗ libx264 not available" && \
    (ffmpeg -hide_banner -encoders | grep libx265 && echo "✓ CPU H.265 (libx265) available") || echo "✗ libx265 not available"