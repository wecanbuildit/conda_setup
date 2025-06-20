# SEAsnake Setup Guide for M-series Mac

This guide provides step-by-step instructions for setting up SEAsnake (RNA-seq pipeline) on Apple Silicon (M1/M2/M3) Macs.

## Prerequisites

Before starting, ensure you have:
- macOS 11.0 (Big Sur) or later
- Administrator access to your Mac
- At least 10GB of free disk space
- Stable internet connection

## Important Notes

- `~` (tilde) represents your home directory (typically `/Users/YOUR_USERNAME/`)
- Replace `YOUR_USERNAME` with your actual Mac username in all commands
- Avoid spaces and special characters in directory names
- If your username contains spaces, escape them with backslash (e.g., `/Users/first\ last/`)
- Check command output: `WARNING` messages are usually okay, but `ERROR` or `FAILURE` require attention

## Step 1: Install Git

1. Check if Git is already installed:
   ```bash
   git --version
   ```

2. If not installed, install via Homebrew (recommended):
   ```bash
   # Install Homebrew first if you don't have it
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   
   # Then install Git
   brew install git
   ```

   Or download from: https://git-scm.com/download/mac

## Step 2: Download SEAsnake

1. Open Terminal (Applications → Utilities → Terminal)

2. Navigate to your desired location (e.g., Desktop):
   ```bash
   cd ~/Desktop
   ```

3. Create and enter SEAsnake directory:
   ```bash
   mkdir SEAsnake
   cd SEAsnake
   ```

4. Clone the repository:
   ```bash
   git clone https://github.com/BIGslu/SEAsnake .
   ```

   Note: The dot (.) at the end clones into the current directory.

## Step 3: Install Miniforge (Recommended for M-series Macs)

For Apple Silicon Macs, we recommend Miniforge instead of Anaconda for better compatibility.

### Why We Recommend Manual Activation for Research

For researchers working on multiple projects, we recommend choosing **"no"** when asked about automatic shell initialization. Here's why:

1. **Project Isolation**: Each research project often requires different package versions. Manual activation ensures you're always aware of which environment is active.

2. **Prevents Cross-contamination**: Avoid accidentally installing packages in the wrong environment or using incorrect package versions.

3. **Cleaner Terminal**: Your command prompt stays normal when working on non-Python tasks, reducing visual clutter.

4. **Explicit Control**: You explicitly choose when to activate conda, making your workflow more intentional and reproducible.

5. **Compatibility**: Some bioinformatics tools have their own Python installations. Manual activation prevents PATH conflicts.

If you later decide you want automatic activation, you can always run `conda init` to enable it.

### Installation Steps

1. Download Miniforge for arm64 (Apple Silicon):
   ```bash
   curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh"
   ```

2. Install Miniforge:
   ```bash
   bash Miniforge3-MacOSX-arm64.sh
   ```
   
   During installation, you'll see these prompts:
   
   - **License Agreement**: Press Enter to review, press Space to scroll, type `yes` to accept
   
   - **Installation Location**: Press Enter to accept default `/Users/YOUR_USERNAME/miniforge3`
     - Or type a custom path if needed
   
   - **Shell Initialization**: 
     ```
     Do you wish to update your shell profile to automatically initialize conda?
     ```
     - **Type `no`** (recommended for research - see explanation above)
     - This means you'll manually activate conda when needed

3. Set up manual activation:
   
   Since you chose not to auto-initialize, you'll see instructions like:
   ```
   To activate conda's base environment in your current shell session:
   eval "$(/Users/YOUR_USERNAME/miniforge3/bin/conda shell.YOUR_SHELL_NAME hook)"
   ```
   
   For M-series Macs (which use zsh by default):
   ```bash
   eval "$(/Users/YOUR_USERNAME/miniforge3/bin/conda shell.zsh hook)"
   ```
   
   Replace `YOUR_USERNAME` with your actual username

4. Verify installation:
   ```bash
   conda --version
   ```
   
   You should see something like:
   ```
   conda 24.3.0
   ```
   
   Note: If you run `which conda`, you might see a shell function definition instead of a simple path. This is normal - conda uses shell functions to manage environment switching.

## Step 4: Install Mamba

Mamba is a faster alternative to conda for package management:

```bash
conda install -n base -c conda-forge mamba -y
```

Verify installation:
```bash
mamba --version
```

## Step 5: Install Bioinformatics Software

### Option 1: Install with Mamba (Recommended)

**Tip:** Open a new Terminal tab (Command+T) to keep your conda session active while checking files.

1. **In a new tab**, check if the environment file exists:
   ```bash
   # Navigate to SEAsnake directory
   cd ~/Desktop/SEAsnake
   
   # Check if the environment file exists
   ls environment/
   ```
   
   **If you see `Hissss_env.yaml`, continue with steps below. If not, skip to Option 2.**

2. Fix M-series compatibility issue in the environment file (still in the new tab):
   ```bash
   # This replaces 'adapterremoval' (incompatible with ARM64) with 'cutadapt'
   sed -i '' 's/- adapterremoval.*/- cutadapt >=5.0/' environment/Hissss_env.yaml
   ```

3. **Switch back to your original Terminal tab** where conda is activated, then create the environment:
   ```bash
   # Make sure you're in the SEAsnake directory
   cd ~/Desktop/SEAsnake
   
   # Create the environment (you'll be prompted to accept package changes)
   mamba env create --name SEAsnake --file environment/Hissss_env.yaml
   ```
   
   When prompted, type `y` to accept the package installation plan.
   
   This may take 10-20 minutes depending on your internet connection.

### Option 2: Manual Installation (If Option 1 Fails)

1. Create a new conda environment:
   ```bash
   conda create --name SEAsnake python=3.9 -y
   conda activate SEAsnake
   ```

2. Configure channels:
   ```bash
   conda config --add channels defaults
   conda config --add channels bioconda
   conda config --add channels conda-forge
   conda config --set channel_priority strict
   ```

3. Install packages individually:
   ```bash
   # Core tools
   mamba install -y fastqc
   mamba install -y bcftools
   mamba install -y samtools
   mamba install -y bwa
   mamba install -y bedtools
   mamba install -y subread
   mamba install -y star
   mamba install -y picard
   mamba install -y snakemake-minimal
   mamba install -y pysam
   mamba install -y cutadapt
   
   # Additional dependencies
   mamba install -y jinja2
   mamba install -y networkx
   mamba install -y graphviz
   mamba install -y matplotlib
   ```

   Note: If any package fails, note the error and continue with the next one.

## Step 6: Verify Installation

1. Activate the environment:
   ```bash
   conda activate SEAsnake
   ```
   
   **Note:** Always use `conda activate` (not `mamba activate`) for switching environments. Mamba is for installing packages, conda is for environment management.

2. Check installed packages:
   ```bash
   conda list
   ```

3. Test a tool:
   ```bash
   fastqc --version
   ```

## Step 7: Install AWS CLI (Optional)

If you need to access data from AWS S3:

```bash
brew install awscli
```

Or download from: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

## Step 8: Environment Management

### Activate environment before use:
```bash
conda activate SEAsnake
```

**Important:** Always use `conda activate/deactivate` for environment switching, even though you installed packages with mamba.

### Deactivate when done:
```bash
conda deactivate
```

### Remove environment (if needed):
```bash
conda env remove --name SEAsnake
```

## Step 9: Troubleshooting Common M-series Mac Issues

### Issue: "Package not available for osx-arm64"

Some packages may not have ARM64 builds. Solutions:

1. Use Rosetta 2 emulation:
   ```bash
   # Create an x86_64 environment
   CONDA_SUBDIR=osx-64 conda create -n SEAsnake_x86 python=3.9
   conda activate SEAsnake_x86
   conda config --env --set subdir osx-64
   ```

2. Then install packages as shown in Option 2 above.

### Issue: SSL Certificate Errors

If you encounter SSL errors:
```bash
conda config --set ssl_verify false
```

Remember to re-enable after installation:
```bash
conda config --set ssl_verify true
```

### Issue: Mamba Not Found

If mamba commands fail:
```bash
# Use full path
~/miniforge3/bin/mamba [command]
```


## Additional Resources

- SEAsnake Documentation: https://github.com/BIGslu/SEAsnake
- Conda Cheat Sheet: https://docs.conda.io/projects/conda/en/latest/user-guide/cheatsheet.html
- Miniforge Documentation: https://github.com/conda-forge/miniforge

## Support

If you encounter issues:
1. Check that you're using the correct environment (`conda activate SEAsnake`)
2. Ensure your macOS is up to date
3. Try using Rosetta 2 for problematic packages
4. Contact the SEAsnake maintainers with detailed error messages

## Uninstallation Guide

To completely remove SEAsnake and all related components from your computer:

### Step 1: Remove SEAsnake Environment
```bash
conda env remove --name SEAsnake
```

If you created an x86_64 environment:
```bash
conda env remove --name SEAsnake_x86
```

### Step 2: Remove SEAsnake Directory
```bash
rm -rf ~/Desktop/SEAsnake
```

### Step 3: Remove Miniforge/Conda (Optional)
If you want to completely remove Miniforge:

1. Remove the installation directory:
   ```bash
   rm -rf ~/miniforge3
   ```

2. Remove conda initialization from your shell (if you chose automatic initialization):
   ```bash
   # For zsh (default on M-series Macs)
   sed -i '' '/# >>> conda initialize >>>/,/# <<< conda initialize <<</d' ~/.zshrc
   
   # For bash (if applicable)
   sed -i '' '/# >>> conda initialize >>>/,/# <<< conda initialize <<</d' ~/.bash_profile
   ```

3. Remove conda configuration directory:
   ```bash
   rm -rf ~/.conda
   rm -rf ~/.condarc
   ```

### Step 4: Remove Homebrew Packages (Optional)
If you installed Git or AWS CLI via Homebrew and want to remove them:
```bash
brew uninstall git
brew uninstall awscli
```

### Step 5: Clean Up Remaining Files
```bash
# Remove any conda package caches
rm -rf ~/Library/Caches/conda
rm -rf ~/.continuum

# Remove mamba cache
rm -rf ~/.mamba
```

After completing these steps, SEAsnake and all related components will be completely removed from your system.

## Automated Setup Script

For a faster, automated installation, you can use the provided setup script instead of following the manual steps above.

### Using the Setup Script

1. Download or clone this repository:
   ```bash
   git clone https://github.com/your-repo/conda_setup.git
   cd conda_setup
   ```

2. Make the script executable (if needed):
   ```bash
   chmod +x setup.sh
   ```

3. Run the setup script:
   ```bash
   ./setup.sh
   ```

### What the Script Does

The `setup.sh` script automates the entire installation process:

- ✅ Checks for Git installation (installs via Homebrew if missing)
- ✅ Downloads and clones the SEAsnake repository
- ✅ Downloads and installs Miniforge for Apple Silicon
- ✅ Fixes ARM64 compatibility issues in the environment file
- ✅ Creates the SEAsnake conda environment with all required packages
- ✅ Installs mamba for faster package management
- ✅ Verifies the installation
- ✅ Creates an activation script (`~/activate_seasnake.sh`) for easy environment switching

### After Running the Script

Once the script completes successfully, you can activate your SEAsnake environment using:

```bash
source ~/activate_seasnake.sh
```

Or manually:
```bash
eval "$(~/miniforge3/bin/conda shell.zsh hook)"
conda activate SEAsnake
```

### Script Requirements

- macOS (optimized for Apple Silicon M-series Macs)
- Administrator access
- Internet connection
- At least 10GB free disk space

### Troubleshooting the Script

If the script fails:
1. Check your internet connection
2. Ensure you have administrator privileges
3. Try running individual commands from the manual installation steps
4. Check the error messages for specific package installation failures