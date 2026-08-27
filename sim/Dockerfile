# pyobs 2.0.1 -- includes the #820 fix: the dummy telescope now publishes
# RA/Dec during mid-slew ticks, so a live reader sees the mount move.
# Python 3.11 because the ecosystem wants >=3.11,<3.14.
FROM python:3.11-slim
ARG PYOBS_VERSION=2.0.1
RUN pip install --no-cache-dir "pyobs-core==${PYOBS_VERSION}"
WORKDIR /work
