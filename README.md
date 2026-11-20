# Lab Core Biowulf

A collection of lightweight Bash scripts to simplify launching interactive computing sessions, RStudio Server, and JupyterLab on the [NIH Biowulf HPC cluster](https://hpc.nih.gov/).

The scripts provide convenient shortcuts for allocating computational resources and starting browser-based environments on Biowulf compute nodes.

## Scripts

| Script | Description |
|---|---|
| `start_sinteractive` | Starts a Slurm interactive session with configurable memory, CPU allocation, and runtime, automatically enabling SSH tunneling. |
| `start_rstudio` | Loads the RStudio Server module and starts RStudio Server on the allocated compute node. |
| `start_jupiter` | Loads the Jupyter module and launches JupyterLab on the allocated compute node. |

## Requirements

- An active NIH Biowulf account.
- SSH access to Biowulf.
- A local web browser.
- An SSH client supporting local port forwarding.
- Access to the RStudio Server or Jupyter modules on Biowulf.

**Important:** Browser-based applications running on Biowulf compute nodes require SSH tunneling. See the [official NIH Biowulf tunneling documentation](https://hpc.nih.gov/docs/tunneling/) before using these scripts.

## Installation

Clone this repository on Biowulf:

```bash
git clone https://github.com/franciscolobo/lab-core-biowulf.git
cd lab-core-biowulf
```

Make the scripts executable:

```bash
chmod +x start_sinteractive start_rstudio start_jupiter
```

You can execute them directly from the repository directory or add the directory to your `PATH`:

```bash
export PATH="$PATH:$HOME/lab-core-biowulf"
```

To make this change persistent, add the export command to your `~/.bashrc` file.

## Usage

### 1. Start an interactive session

Connect to Biowulf:

```bash
ssh username@biowulf.nih.gov
```

Start an interactive session using `start_sinteractive`:

```bash
start_sinteractive -m 32g -c 4 -t 04:00:00
```

**Parameters**

| Parameter | Description | Example |
|---|---|---|
| `-m` | Requested memory | `32g` |
| `-c` | Number of CPUs per task | `4` |
| `-t` | Maximum wall time (`HH:MM:SS`) | `04:00:00` |

The script executes:

```bash
sinteractive \
    --gres=lscratch:200 \
    --mem=32g \
    --cpus-per-task=4 \
    --time=04:00:00 \
    --tunnel
```

**Notes:**

- All three parameters are required.
- The current script requests 200 GB of local scratch storage (`lscratch`) by default.
- SSH tunneling is enabled automatically using `--tunnel`.
- The allocated resources and session duration are subject to Biowulf's scheduling policies and resource availability.

### 2. Establish an SSH tunnel

After the interactive session starts, Biowulf displays a command to establish an SSH tunnel from your local computer to the allocated compute node.

For example:

```bash
ssh -L 45000:localhost:45000 username@biowulf.nih.gov
```

**Open a separate terminal on your local computer** and execute the exact command provided by Biowulf.

Keep this terminal open while using RStudio or JupyterLab.

The tunnel connects your browser to the application running on the compute node without exposing the application directly to the network.

**Important:** Port `45000` is only an example. Always use the port assigned to your interactive session.

For additional instructions, including Windows setup, consult:

[NIH Biowulf — SSH Tunneling](https://hpc.nih.gov/docs/tunneling/)

### 3A. Launch RStudio Server

Inside the interactive session on the compute node, execute:

```bash
start_rstudio
```

The script runs:

```bash
module load rstudio-server
rstudio-server
```

RStudio Server will display a local URL and, if required, login credentials.

Open the provided URL in your local browser once the SSH tunnel is established.

For example:

```text
http://localhost:45000/
```

Use the actual URL and authentication information displayed by RStudio Server.

For additional information, see the [Biowulf RStudio Server documentation](https://hpc.nih.gov/apps/rstudio-server.html).

### 3B. Launch JupyterLab

Alternatively, inside the interactive session, execute:

```bash
start_jupiter
```

The script runs:

```bash
module load jupyter
jupyter lab --ip localhost --port $PORT1 --no-browser
```

The environment variable `PORT1` is assigned automatically by Biowulf when the interactive session is started with `--tunnel`.

JupyterLab displays a URL containing the assigned port and an authentication token.

Open this URL in your local browser after establishing the SSH tunnel.

For example:

```text
http://localhost:45000/lab?token=YOUR_TOKEN
```

Replace the example URL with the actual URL printed by JupyterLab.

For additional information, see the [Biowulf Jupyter documentation](https://hpc.nih.gov/apps/jupyter.html).

## Example workflow

A complete example of launching JupyterLab:

**Terminal 1 — Connect to Biowulf and allocate resources**

```bash
ssh username@biowulf.nih.gov
start_sinteractive -m 64g -c 8 -t 08:00:00
```

Wait until Slurm allocates a compute node.

**Terminal 2 — Create the SSH tunnel from your local computer**

```bash
ssh -L 45000:localhost:45000 username@biowulf.nih.gov
```

Use the actual port assigned by Biowulf.

**Terminal 1 — Start JupyterLab on the compute node**

```bash
start_jupiter
```

**Local web browser**

Open the URL printed by JupyterLab.

To use RStudio instead, replace `start_jupiter` with `start_rstudio`.

## Ending a session

When finished:

1. Save your work in RStudio or JupyterLab.
2. Stop the application using `Ctrl+C` in the terminal where it is running, when applicable.
3. Exit the interactive session using `exit`.
4. Close the local SSH tunnel.

Interactive jobs consume allocated cluster resources, so terminate sessions when they are no longer needed.

## Troubleshooting

**Browser cannot connect to localhost**

Verify that the local SSH tunnel is running and that the browser URL uses the port assigned by Biowulf.

**JupyterLab starts but cannot be accessed**

Confirm that JupyterLab is running on `$PORT1`:

```bash
echo $PORT1
```

**SSH tunnel disconnected**

On Biowulf, run:

```bash
reconnect_tunnels
```

This prints the SSH command needed to reconnect to any active tunnels.

**Slurm job remains pending**

The requested memory, CPU count, or scratch capacity may not currently be available. Check the job's status or consider requesting fewer resources.

## Additional resources

- [NIH Biowulf](https://hpc.nih.gov/)
- [Biowulf User Guide](https://hpc.nih.gov/docs/userguide.html)
- [SSH Tunneling](https://hpc.nih.gov/docs/tunneling/)
- [RStudio Server on Biowulf](https://hpc.nih.gov/apps/rstudio-server.html)
- [Jupyter on Biowulf](https://hpc.nih.gov/apps/jupyter.html)

## Disclaimer

These are personal convenience scripts and are not official NIH Biowulf utilities. Users should consult the current NIH HPC documentation and follow applicable institutional policies.
