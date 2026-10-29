# vLLM v0.32.0 + the GLM-5.x MTP head name fix (same change as vllm-project/vllm#27193, in the v0.32.0 file),
# from corticph/vllm branch glm-mtp-head-fix-pla-7563.
# vLLM names the MTP layer's output head model.layers.78.head, so it misses the checkpoint's
# model.layers.78.shared_head entry in the quantization ignore list and loads the head as NVFP4.
# Only one Python file changes, so the patched file is copied over the release image instead of rebuilding vLLM.
# Build context: patch/ holds this branch's vllm/models/deepseek_v32/nvidia/mtp.py.
FROM corti.azurecr.io/vllm/vllm:v0.32.0

COPY patch/mtp.py /tmp/vllm-patch/

# Refuse to patch anything but the exact v0.32.0 file, then check the copy landed where vLLM loads it from.
RUN set -eu; \
    target="$(python3 -c 'import importlib.util; print(importlib.util.find_spec("vllm").submodule_search_locations[0])')/models/deepseek_v32/nvidia"; \
    echo "vLLM GLM MTP module: $target"; \
    echo "7015e365cc06877e0b86ec1659ef601817b063667d13a2c45dc0602187d54f93  $target/mtp.py" | sha256sum -c -; \
    cp /tmp/vllm-patch/mtp.py "$target/mtp.py"; \
    rm -f "$target"/__pycache__/mtp.*.pyc; \
    echo "a37848f21abdba23170771e8187ee341260890c3f1b88bd2d6ab0d5f5ee7fdce  $target/mtp.py" | sha256sum -c -; \
    rm -rf /tmp/vllm-patch; \
    mkdir -p /opt/corti; \
    echo "vllm v0.32.0 + GLM MTP head name fix (vllm-project/vllm#27193), corticph/vllm branch glm-mtp-head-fix-pla-7563" > /opt/corti/vllm-patch-info.txt
