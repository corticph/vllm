# vLLM v0.33.0 + vllm-project/vllm#61530 (MXFP4-packed Engram tables for DeepSeek-V4.1), not yet merged upstream.
# Resolved by hand against the peer HBM reads added in v0.33.0: a packed table that would use them raises at startup.
# From corticph/vllm branch deepseek-v41-engram-mxfp4-v0.33.0-pr.61530.
# Only one Python file changes, so the patched file is copied over the release image instead of rebuilding vLLM.
# Build context: patch/engram.py.
FROM corti.azurecr.io/vllm/vllm:v0.33.0@sha256:d825baf9ce4b1b6bfc5ecc7b79880c7e6748c65a72d697ce3d47607aafa84c6e

COPY patch/engram.py /tmp/vllm-patch/

# Refuse to patch anything but the exact v0.33.0 file, then check the copy landed where vLLM loads it from.
RUN set -eu; \
    target="$(python3 -c 'import importlib.util; print(importlib.util.find_spec("vllm").submodule_search_locations[0])')/models/deepseek_v41/common"; \
    echo "vLLM Engram module: $target"; \
    echo "720923ccd796dfdeab4b396eeeea79cd4c11a9dcd2e6ef8d57c0390fa4cf7ab2  $target/engram.py" | sha256sum -c -; \
    cp /tmp/vllm-patch/engram.py "$target/engram.py"; \
    rm -f "$target"/__pycache__/engram.*.pyc; \
    echo "33d30d77b597a84f6655660871d4f90291e630d107a2d4378d91054a5fa91907  $target/engram.py" | sha256sum -c -; \
    rm -rf /tmp/vllm-patch; \
    mkdir -p /opt/corti; \
    echo "vllm v0.33.0 + MXFP4 Engram (vllm-project/vllm#61530), corticph/vllm branch deepseek-v41-engram-mxfp4-v0.33.0-pr.61530" > /opt/corti/vllm-patch-info.txt
