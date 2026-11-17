# vLLM v0.34.0 + vllm-project/vllm#65340 (GDN crash when adaptive verification trims a mixed prefill and verification
# batch, still open upstream at commits c1d8744d2 and c96d2ac9a), for GLM-5.3-Flash, from corticph/vllm branch
# glm53-flash-v0.34.0-pla-8143.
# v0.34.0 already includes vllm-project/vllm#60229, so only gdn_attn.py changes.
# Only a Python file changes, so the patched file is copied over the release image instead of rebuilding vLLM.
# Build context: patch/gdn_attn.py.
FROM corti.azurecr.io/vllm/vllm:v0.34.0@sha256:20fdb089454afbb324efa4110a2c025ff7b27cc1a086e07aa580b18df8ad1ce1

LABEL org.opencontainers.image.description="vLLM v0.34.0 + vllm#65340 (c1d8744d2, c96d2ac9a), for GLM-5.3-Flash" \
      org.opencontainers.image.source="https://github.com/corticph/vllm/tree/glm53-flash-v0.34.0-pla-8143" \
      org.opencontainers.image.base.name="vllm/vllm:v0.34.0"

COPY patch/gdn_attn.py /tmp/vllm-patch/

# Refuse to patch anything but the exact v0.34.0 file, then check the copy landed where vLLM loads it from.
RUN set -eu; \
    vllm_dir="$(python3 -c 'import importlib.util; print(importlib.util.find_spec("vllm").submodule_search_locations[0])')"; \
    echo "vLLM package: $vllm_dir"; \
    patch_file() { \
        src="$1"; dest="$vllm_dir/$2"; before="$3"; after="$4"; \
        echo "$before  $dest" | sha256sum -c -; \
        cp "/tmp/vllm-patch/$src" "$dest"; \
        rm -f "$(dirname "$dest")"/__pycache__/"$(basename "$dest" .py)".*.pyc; \
        echo "$after  $dest" | sha256sum -c -; \
        python3 -c 'import py_compile, sys; py_compile.compile(sys.argv[1], doraise=True)' "$dest"; \
    }; \
    patch_file gdn_attn.py v1/attention/backends/gdn_attn.py \
        09abe3f04db0b8e422daa4ef7d15eb6c1347fca75c9988309bb32f6cbbf87183 \
        454dbcec4599280a4220bd5126394da5948dd5c873e456a124517ab1c403f577; \
    python3 -c 'import vllm; assert vllm.__version__.startswith("0.34.0"), vllm.__version__; print("vllm", vllm.__version__)'; \
    grep -q 'output_size=int(query_lens.sum().item())' "$vllm_dir/v1/attention/backends/gdn_attn.py"; \
    rm -rf /tmp/vllm-patch; \
    mkdir -p /opt/corti; \
    echo "vllm v0.34.0 + vllm-project/vllm#65340 (c1d8744d2, c96d2ac9a), GLM-5.3-Flash, corticph/vllm branch glm53-flash-v0.34.0-pla-8143" > /opt/corti/vllm-patch-info.txt
