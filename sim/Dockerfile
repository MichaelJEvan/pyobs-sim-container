# pyobs 2.0.1 -- includes the #820 fix: the dummy telescope now publishes
# RA/Dec during mid-slew ticks, so a live reader sees the mount move.
# Python 3.11 because the ecosystem wants >=3.11,<3.14.
FROM python:3.11-slim
ARG PYOBS_VERSION=2.1.0

# [full], not bare pyobs-core. A default install cannot take an exposure:
# DummyCamera._simulate_image imports photutils to paint its star field, and
# photutils only arrives with the extra -- so the first grab fails with
# "No module named 'photutils'" and no hint that an extra exists. Measured
# 2026-08-30 from the GUI.
#
# It is not just the camera. Seven of the twelve packages in [full] are
# imported by real pyobs code, all of it inside functions rather than at the
# top of the file, so every module loads cleanly and only breaks when you
# press the button. Aperture photometry and source detection (photutils,
# sep), calibration frames (ccdproc) and the focus series (lmfit) are the
# whole science half of this observatory.
#
# The compiler is here for exactly one package. photutils 3.0.0 ships wheels
# for macOS, Windows and *Intel* Linux and none for arm64 Linux, so on an
# Apple Silicon host it has to be built from source -- and slim has no cc.
# Every other package in [full] has an aarch64 wheel; checked one by one
# rather than assumed. gcc goes in and comes out inside a single layer, so
# the image does not carry a toolchain it will never use again.
RUN apt-get update \
 && apt-get install -y --no-install-recommends gcc g++ \
 && pip install --no-cache-dir "pyobs-core[full]==${PYOBS_VERSION}" \
 && apt-get purge -y --auto-remove gcc g++ \
 && rm -rf /var/lib/apt/lists/*
WORKDIR /work
