# RPI-SYSTEMD-CONFIG(1)

## NAME
rpi-systemd-config - Query systemd configuration files

## SYNOPSIS
**rpi-systemd-config** [**\-\-version**] *NAME* [*VARIABLE* *KEY*]...

## DESCRIPTION
**rpi-systemd-config** extracts values from systemd configuration files and outputs shell variable assignments. It uses systemd's own configuration parsing logic, following the same precedence rules and drop-in directory support as systemd itself.

Supports both Debian Bookworm (systemd 252) and Debian Trixie (systemd 257).

The tool searches configuration files using systemd's standard directory hierarchy and supports drop-in configuration files as described in **systemd.unit**(5).

## ARGUMENTS
*NAME*
: Configuration file name relative to the systemd configuration prefix (e.g., **systemd/system.conf**, **systemd/logind.conf**).

*VARIABLE* *KEY*
: Pairs of shell variable names and configuration keys. Each *KEY* must be in the format **section::key**.

## OPTIONS
**\-v**, **\-\-version**
: Show version information and exit.

**\-h**, **\-\-help**
: Show help message and exit.

## CONFIGURATION FILE LOOKUP
**rpi-systemd-config** follows systemd's standard lookup order:

1. **/etc/**_NAME_ - Local administrator configuration
2. **/run/**_NAME_ - Runtime configuration  
3. **/usr/local/lib/**_NAME_ - Local package configuration
4. **/usr/lib/**_NAME_ - Distribution package configuration

Drop-in files from corresponding **.d** directories are processed following the same precedence rules.

## OUTPUT FORMAT
For each configuration entry found, outputs a shell variable assignment:

```
VARIABLE='value'
```

Values are properly escaped for shell consumption. Missing entries produce no output.

## EXAMPLES
Use in shell scripts:
```bash
#!/bin/bash
eval "$(rpi-systemd-config systemd/system.conf \
    DEFAULT_TIMEOUT Manager::DefaultTimeoutStopSec \
    DEFAULT_RESTART Manager::DefaultRestartSec)"

if [ -n "${DEFAULT_TIMEOUT:-}" ]; then
    echo "System default timeout is configured: $DEFAULT_TIMEOUT"
else
    echo "Using systemd built-in default timeout"
fi
```

## EXIT STATUS
Exits with status 0 on success. Missing configuration entries do not cause non-zero exit status.

## COMPARISON WITH SYSTEMD-ANALYZE
**systemd-analyze cat-config** shows complete file contents with comments, whilst **rpi-systemd-config** extracts specific values for programmatic use:

```bash
$ rpi-systemd-config systemd/system.conf TIMEOUT Manager::DefaultTimeoutStopSec
TIMEOUT='30s'
```

## LIMITATIONS
- Only supports configuration files that use systemd's standard configuration parser
- Requires access to systemd's internal shared library
- Supports systemd versions 252 and 257
- **Value parsing**: Uses systemd's generic string parser (`config_parse_string`) only. Does not perform specialized validation or parsing that systemd uses internally (such as `config_parse_bool`, `config_parse_hw_addr`, `config_parse_percent`, etc.). Values are returned as raw strings exactly as they appear in configuration files, without type validation or conversion.

## SEE ALSO
**systemd**(1), **systemd-analyze**(1), **systemd.syntax**(7), **systemd.unit**(5), **apt-config**(8)

## NOTES
**rpi-systemd-config** uses systemd's own configuration parsing functions to ensure compatibility. The output format is designed for consumption by shell **eval** statements, similar to **apt-config**(8).
