# ioc-calc-autosave

Generic IOC container image with calculation records and autosave support.

## Container Image

```
ghcr.io/khesse-757/ioc-calc-autosave-runtime:0.1.0
```

## Support Modules Included

| Module | Version | Purpose |
|--------|---------|---------|
| EPICS Base | 7.0.9 | Core EPICS |
| iocStats | - | IOC health monitoring |
| pvlogging | - | PV logging support |
| calc | - | Calculation records (calc, calcout, etc.) |
| autosave | - | Save/restore PV values across restarts |

## Using This Image

### In a Services Repository

```yaml
# services/my-ioc/compose.yml
services:
  my-ioc:
    image: ghcr.io/khesse-757/ioc-calc-autosave-runtime:0.1.0
    # ... rest of config
```

### Autosave Configuration

1. Add volume for persistence in `compose.yml`:
   ```yaml
   volumes:
     - autosave-my-ioc:/autosave
   
   # At bottom of file:
   volumes:
     autosave-my-ioc:
   ```

2. Configure in `ioc.yaml`:
   ```yaml
   entities:
     - type: epics.StartupCommand
       command: |
         set_requestfile_path("/epics/ioc", "config")

     - type: autosave.Autosave
       P: "MY:"
       path: /autosave
       positions_req_period: 0
       settings_req_period: 10
   ```

3. Create `config/autosave_settings.req`:
   ```
   MY:SETPOINT
   MY:GAIN
   MY:OFFSET
   ```

## Available Entity Types

### From autosave module

| Entity | Required Parameters | Description |
|--------|---------------------|-------------|
| `autosave.Autosave` | `P` (PV prefix) | Configure autosave |

Key parameters:
- `P`: PV prefix for status records (must be quoted if contains colon: `"MY:"`)
- `path`: Save file location (default: `/autosave`)
- `settings_req_period`: Seconds between saves (default: 30)
- `positions_req_period`: Seconds between position saves (default: 5, use 0 to disable)

### From devIocStats module

| Entity | Required Parameters | Description |
|--------|---------------------|-------------|
| `devIocStats.iocAdminSoft` | `IOC` | IOC statistics PVs |

### From epics module

| Entity | Required Parameters | Description |
|--------|---------------------|-------------|
| `epics.EpicsEnvSet` | `name`, `value` | Set environment variable |
| `epics.StartupCommand` | `command` | Pre-iocInit command |
| `epics.PostStartupCommand` | `command` | Post-iocInit command |
| `epics.dbpf` | `pv`, `value` | Set PV value at startup |

## Building Locally

```bash
# Clone
git clone https://github.com/khesse-757/ioc-calc-autosave.git
cd ioc-calc-autosave

# Build (Mac with Apple Silicon)
docker build --platform linux/amd64 -t ioc-calc-autosave:local .

# Build (Linux)
docker build -t ioc-calc-autosave:local .
```

Build time: ~5-15 minutes (compiles EPICS base and support modules)

### Using Local Build

```yaml
# In your services compose.yml
image: ioc-calc-autosave:local
```

## Adding Support Modules

1. Check available modules:
   ```bash
   ls ibek-support/
   ```

2. Edit `Dockerfile` to add module:
   ```dockerfile
   COPY ibek-support/asyn/ asyn
   RUN ansible.sh asyn
   ```

3. Rebuild:
   ```bash
   docker build --platform linux/amd64 -t ioc-calc-autosave:local .
   ```

4. Tag and push for CI:
   ```bash
   git add -A
   git commit -m "Add asyn support"
   git tag 0.2.0
   git push origin main --tags
   ```

## CI/CD

GitHub Actions automatically:
1. Builds container image on every push
2. Pushes to ghcr.io on tagged releases
3. Creates GitHub Release with ibek schema

Images are published to:
- `ghcr.io/khesse-757/ioc-calc-autosave-runtime:<tag>`
- `ghcr.io/khesse-757/ioc-calc-autosave-developer:<tag>`

## Schema for IDE Support

Add to `ioc.yaml` for VS Code autocomplete:
```yaml
# yaml-language-server: $schema=https://github.com/khesse-757/ioc-calc-autosave/releases/download/0.1.0/ibek.ioc.schema.json
```

## Developer Container

This repository includes VS Code devcontainer configuration. Open in VS Code and select "Reopen in Container" for a full development environment.

See: https://epics-containers.github.io/main/tutorials/dev_container.html

## Template Info

Generated from [ioc-template](https://github.com/epics-containers/ioc-template).

Update to latest template:
```bash
pip install copier
copier update --trust .
```

## Links

- [t01-services](https://github.com/khesse-757/t01-services) - Example services repo using this image
- [epics-containers documentation](https://epics-containers.github.io/main/)
- [ibek-support](https://github.com/epics-containers/ibek-support) - Support module recipes