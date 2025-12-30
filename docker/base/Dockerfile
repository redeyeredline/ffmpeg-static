# syntax=docker/dockerfile:1

# Build FFmpeg base version (CPU + QuickSync only) for Cliprr
FROM ubuntu:22.04 AS builder

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive

# Install build dependencies
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
    libjansson-dev \
    liblzma-dev \
    libmp3lame-dev \
    libnuma-dev \
    libogg-dev \
    libopus-dev \
    libsamplerate0-dev \
    libspeex-dev \
    libtheora-dev \
    libtool \
    libtool-bin \
    libturbojpeg0-dev \
    libvorbis-dev \
    libx264-dev \
    libxml2-dev \
    libvpx-dev \
    m4 \
    make \
    meson \
    nasm \
    ninja-build \
    patch \
    pkg-config \
    tar \
    zlib1g-dev \
    yasm \
    wget \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Create build directory
WORKDIR /tmp/ffmpeg-build

# Download FFmpeg source
ARG FFMPEG_VERSION=7.1.1
RUN wget https://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.bz2 && \
    tar -xf ffmpeg-${FFMPEG_VERSION}.tar.bz2 && \
    cd ffmpeg-${FFMPEG_VERSION}

# Configure FFmpeg without NVIDIA support (CPU + QuickSync only)
WORKDIR /tmp/ffmpeg-build/ffmpeg-${FFMPEG_VERSION}
RUN ./configure \
    --prefix=/usr/local \
    --enable-gpl \
    --enable-nonfree \
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
    --enable-postproc \
    --enable-pthreads \
    --enable-shared

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

# Install minimal runtime dependencies (matching what's actually needed)
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
    libva-glx2 \
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
