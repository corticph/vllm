# vLLM v0.33.0 + vllm-project/vllm#58209 (defer the disposable DeepseekV32 MTP head), nvidia file only,
# from corticph/vllm branch glm-mtp-defer-head-v0.33.0-pla-7563.
# Stock v0.33.0 builds the GLM-5.x MTP head as NVFP4 and crashes at startup on the Inferact checkpoint:
# "NVFP4 weight_scale for layer 'parallel_lm_head' was never loaded (still NaN)".
# Only one Python file changes, so the patched file is copied over the release image instead of rebuilding vLLM.
# Build context: patch/mtp.py.
FROM corti.azurecr.io/vllm/vllm:v0.33.0@sha256:d825baf9ce4b1b6bfc5ecc7b79880c7e6748c65a72d697ce3d47607aafa84c6e

COPY patch/mtp.py /tmp/vllm-patch/

# Refuse to patch anything but the exact v0.33.0 file, then check the copy landed where vLLM loads it from.
RUN set -eu; \
    target="$(python3 -c 'import importlib.util; print(importlib.util.find_spec("vllm").submodule_search_locations[0])')/models/deepseek_v32/nvidia"; \
    echo "vLLM GLM MTP module: $target"; \
    echo "dbfff5ca8c3aea2961d5a466d1bb7ac68ba58cf6c770808d80ed54e1f85a5b21  $target/mtp.py" | sha256sum -c -; \
    cp /tmp/vllm-patch/mtp.py "$target/mtp.py"; \
    rm -f "$target"/__pycache__/mtp.*.pyc; \
    echo "550a4d58860d61c1e6f848cee12f47c024535afd9642f68310475a0cbaea310f  $target/mtp.py" | sha256sum -c -; \
    rm -rf /tmp/vllm-patch; \
    mkdir -p /opt/corti; \
    echo "vllm v0.33.0 + GLM MTP deferred head (vllm-project/vllm#58209), nvidia file only, corticph/vllm branch glm-mtp-defer-head-v0.33.0-pla-7563" > /opt/corti/vllm-patch-info.txt
