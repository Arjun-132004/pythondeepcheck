FROM python:3.14-slim AS build
WORKDIR /app
# PEP 517 build: pip reads pyproject.toml, resolves the declared backend
# (poetry-core, hatchling, pdm-backend, setuptools) and installs the project
# with its dependencies. No packaging tool needs to be baked into the image.
COPY . .
RUN pip install --no-cache-dir --prefix=/install .

FROM python:3.14-slim AS runtime
WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
COPY --from=build /install /usr/local
COPY . .
# Unprivileged runtime user; nothing in the image needs to be written to.
# USER must be a NUMERIC uid: Kubernetes' runAsNonRoot check refuses to
# start a container whose USER is a name, because it cannot verify from
# outside the image that the name is non-root (2026-09, java-app-backend).
RUN useradd --create-home --uid 10001 appuser && chown -R appuser /app
USER 10001
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=3s --start-period=20s \
  CMD python -c "import urllib.request,sys; sys.exit(0 if urllib.request.urlopen('http://127.0.0.1:8000/').status==200 else 1)"
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
