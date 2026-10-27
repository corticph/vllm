# vLLM v0.32.0 + vllm-project/vllm#57128 (Mamba prefix-cache lookup honours drop_eagle_block),
# from corticph/vllm branch deploy-vllm-mamba-cache-manager-fix-pla-7694 at 2719a9dea.
# Only Python files change, so the patched files are copied over the release image instead of rebuilding vLLM.
# Build context: patch/ holds this branch's vllm/v1/core/{single_type_kv_cache_manager,kv_cache_coordinator}.py
# and tests/ holds tests/v1/core/test_single_type_kv_cache_manager.py.
FROM corti.azurecr.io/vllm/vllm:v0.32.0 AS patched

COPY patch/single_type_kv_cache_manager.py patch/kv_cache_coordinator.py /tmp/vllm-patch/

# Refuse to patch anything but the exact v0.32.0 files, then check the copies landed where vLLM loads them from.
RUN set -eu; \
    core="$(python3 -c 'import importlib.util; print(importlib.util.find_spec("vllm").submodule_search_locations[0])')/v1/core"; \
    echo "vLLM core: $core"; \
    echo "fe38816a388bb5b615231f621adc208e3893d20349865d041cc4f646d4d89b0a  $core/single_type_kv_cache_manager.py" | sha256sum -c -; \
    echo "efd7193391197c555ebc0c8f2b593a6c4cb5b7da2978cd57414c0b71170f5971  $core/kv_cache_coordinator.py" | sha256sum -c -; \
    cp /tmp/vllm-patch/*.py "$core/"; \
    rm -f "$core"/__pycache__/single_type_kv_cache_manager.*.pyc "$core"/__pycache__/kv_cache_coordinator.*.pyc; \
    echo "457dd1f6f19d45305cfa3f202a911371876293877cf78f1774ad324bba7188b9  $core/single_type_kv_cache_manager.py" | sha256sum -c -; \
    echo "3c508c4c2b70ab1ddd4c8f299f42a5897b1fa60bd204a8bbaac2890dc56ac0b8  $core/kv_cache_coordinator.py" | sha256sum -c -; \
    rm -rf /tmp/vllm-patch

# Throwaway stage: run the KV cache manager tests (including #57128's) against the patched install.
FROM patched AS test
COPY tests/test_single_type_kv_cache_manager.py /tmp/vllm-tests/
RUN set -eu; \
    python3 -c 'import pytest' 2>/dev/null || python3 -m pip install --no-cache-dir pytest; \
    cd /tmp/vllm-tests; \
    python3 -m pytest -q -p no:cacheprovider test_single_type_kv_cache_manager.py > result.txt 2>&1 \
        || { cat result.txt; exit 1; }; \
    cat result.txt; \
    echo "vllm v0.32.0 + vllm-project/vllm#57128, corticph/vllm@2719a9dea" > /tmp/vllm-patch-info.txt; \
    tail -1 /tmp/vllm-tests/result.txt >> /tmp/vllm-patch-info.txt

# Final image: the patched install plus a small record of what was applied and the test result.
# Copying from the test stage makes the build fail when the tests fail.
FROM patched
COPY --from=test /tmp/vllm-patch-info.txt /opt/corti/vllm-patch-info.txt
