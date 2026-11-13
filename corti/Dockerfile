# vLLM v0.33.0 + vllm-project/vllm#60229 (adaptive verification on GLM-5.3-Flash) + vllm-project/vllm#65340 (GDN
# crash when adaptive verification trims a mixed prefill and verification batch, still open upstream at commits
# c1d8744d2 and c96d2ac9a), for GLM-5.3-Flash only, from corticph/vllm branch glm53-flash-av-v0.33.0-pla-8096.
# Stock v0.33.0 refuses adaptive verification on GLM-5.3-Flash at startup: its KDA layers use the GDN state-space
# backend, which v0.33.0 does not let adaptive verification trim.
# Only Python files change, so the patched files are copied over the release image instead of rebuilding vLLM.
# Build context: patch/gdn_attn.py, patch/sparse_indexer.py, patch/indexer.py.
FROM corti.azurecr.io/vllm/vllm:v0.33.0@sha256:d825baf9ce4b1b6bfc5ecc7b79880c7e6748c65a72d697ce3d47607aafa84c6e

LABEL org.opencontainers.image.description="vLLM v0.33.0 + vllm#60229 + vllm#65340 (c1d8744d2, c96d2ac9a), for GLM-5.3-Flash only" \
      org.opencontainers.image.source="https://github.com/corticph/vllm/tree/glm53-flash-av-v0.33.0-pla-8096" \
      org.opencontainers.image.base.name="vllm/vllm:v0.33.0"

COPY patch/gdn_attn.py patch/sparse_indexer.py patch/indexer.py /tmp/vllm-patch/

# Refuse to patch anything but the exact v0.33.0 files, then check each copy landed where vLLM loads it from.
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
        90f67d9fa8b06fd531579ed1198678c3251baabb47a7b0c9dfdc828d4361ec5e \
        7676038db4c3ccafad9f6f9aa14424cd7280b578667e3f4419c2770138c1c291; \
    patch_file sparse_indexer.py models/glm5next/common/sparse_indexer.py \
        a3ab1edda8490b8e21c1c240e07e8c8fcd0bb34246a9ed1f64acfe067d15067c \
        5fe24be2034575422585889921f70d57c7ca64466559bc7ac4895e8adda5ebdb; \
    patch_file indexer.py v1/attention/backends/mla/indexer.py \
        6457fb5a755ca53addce7731cb56770493371ad9f3728251e7ff33515b67965c \
        0f490404c8be944d9cc37a5400b6e14d2a7cbc579bf56e5e2715d7222b653908; \
    rm -rf /tmp/vllm-patch; \
    mkdir -p /opt/corti; \
    echo "vllm v0.33.0 + vllm-project/vllm#60229 + vllm-project/vllm#65340 (c1d8744d2, c96d2ac9a), GLM-5.3-Flash only, corticph/vllm branch glm53-flash-av-v0.33.0-pla-8096" > /opt/corti/vllm-patch-info.txt
