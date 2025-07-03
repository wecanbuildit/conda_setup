#!/bin/zsh

# SEAsnake Setup Script for M-series Mac
# This script automates the installation process described in README.md

set -e  # Exit on any error

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Function to print colored output
print_status() {
    echo -e "${BLUE}[INFO]${NC} $1"
}

print_success() {
    echo -e "${GREEN}[SUCCESS]${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1"
}

print_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Function to check if command exists
command_exists() {
    command -v "$1" >/dev/null 2>&1
}

# Function to prompt user
prompt_user() {
    while true; do
        read "yn?$1 (y/n): "
        case $yn in
            [Yy]* ) return 0;;
            [Nn]* ) return 1;;
            * ) echo "Please answer yes or no.";;
        esac
    done
}

print_status "Starting SEAsnake setup for M-series Mac..."

# Check if running on macOS
if [[ "$OSTYPE" != "darwin"* ]]; then
    print_error "This script is designed for macOS only."
    exit 1
fi

# Check if running on Apple Silicon
if [[ $(uname -m) != "arm64" ]]; then
    print_warning "This script is optimized for Apple Silicon (M-series) Macs."
    print_warning "You may encounter compatibility issues on Intel Macs."
    if ! prompt_user "Do you want to continue anyway?"; then
        exit 0
    fi
fi

# Step 1: Check/Install Git
print_status "Checking Git installation..."
if command_exists git; then
    print_success "Git is already installed: $(git --version)"
else
    print_status "Git not found. Checking for Homebrew..."
    if command_exists brew; then
        print_status "Installing Git via Homebrew..."
        brew install git
    else
        print_warning "Homebrew not found. Please install Git manually from https://git-scm.com/download/mac"
        print_warning "Or install Homebrew first: /bin/bash -c \"\$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)\""
        exit 1
    fi
fi

# Step 2: Setup SEAsnake directory
print_status "Setting up SEAsnake directory..."
SEASNAKE_DIR="$HOME/Desktop/SEAsnake"

if [[ -d "$SEASNAKE_DIR" ]]; then
    print_warning "SEAsnake directory already exists at $SEASNAKE_DIR"
    if prompt_user "Do you want to remove it and start fresh?"; then
        rm -rf "$SEASNAKE_DIR"
        print_success "Removed existing directory"
    else
        print_status "Using existing directory"
    fi
fi

if [[ ! -d "$SEASNAKE_DIR" ]]; then
    mkdir -p "$SEASNAKE_DIR"
    print_status "Cloning SEAsnake repository..."
    git clone https://github.com/BIGslu/SEAsnake "$SEASNAKE_DIR"
    print_success "SEAsnake repository cloned to $SEASNAKE_DIR"
fi

# Step 3: Install Miniforge
print_status "Checking Miniforge installation..."
MINIFORGE_PATH="$HOME/miniforge3"

if [[ -d "$MINIFORGE_PATH" ]]; then
    print_success "Miniforge already installed at $MINIFORGE_PATH"
else
    print_status "Downloading Miniforge for Apple Silicon..."
    cd /tmp
    curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh"
    
    print_status "Installing Miniforge..."
    print_warning "During installation:"
    print_warning "- Press ENTER to review license, SPACE to scroll, type 'yes' to accept"
    print_warning "- Press ENTER to accept default installation location"
    print_warning "- Type 'no' when asked about shell initialization (recommended for research)"
    
    bash Miniforge3-MacOSX-arm64.sh
    
    # Clean up installer
    rm -f Miniforge3-MacOSX-arm64.sh
fi

# Step 4: Setup conda environment
print_status "Setting up conda environment..."

# Source conda for zsh
if [[ -f "$MINIFORGE_PATH/bin/conda" ]]; then
    eval "$($MINIFORGE_PATH/bin/conda shell.zsh hook)"
else
    print_error "Miniforge installation not found at expected location"
    exit 1
fi

# Install mamba if not already installed
print_status "Installing mamba..."
conda install -n base -c conda-forge mamba -y

# Step 5: Create SEAsnake environment
print_status "Creating SEAsnake environment..."
cd "$SEASNAKE_DIR"

# Check if environment file exists and fix ARM64 compatibility
if [[ -f "environment/Hissss_env.yaml" ]]; then
    print_status "Fixing ARM64 compatibility in environment file..."
    sed -i '' 's/- adapterremoval.*/- cutadapt >=5.0/' environment/Hissss_env.yaml
    
    print_status "Creating environment from YAML file..."
    mamba env create --name SEAsnake --file environment/Hissss_env.yaml
else
    print_warning "Environment file not found. Creating environment manually..."
    
    # Create base environment
    conda create --name SEAsnake python=3.9 -y
    
    # Activate environment
    conda activate SEAsnake
    
    # Configure channels
    conda config --add channels defaults
    conda config --add channels bioconda
    conda config --add channels conda-forge
    conda config --set channel_priority strict
    
    # Install packages
    print_status "Installing bioinformatics packages..."
    packages=(
        "fastqc"
        "bcftools"
        "samtools"
        "bwa"
        "bedtools"
        "subread"
        "star"
        "picard"
        "snakemake-minimal"
        "pysam"
        "cutadapt"
        "jinja2"
        "networkx"
        "graphviz"
        "matplotlib"
    )
    
    for package in "${packages[@]}"; do
        print_status "Installing $package..."
        if ! mamba install -y "$package"; then
            print_warning "Failed to install $package, continuing..."
        fi
    done
fi

# Step 6: Verify installation
print_status "Verifying installation..."
conda activate SEAsnake

if fastqc --version >/dev/null 2>&1; then
    print_success "FastQC installed successfully: $(fastqc --version)"
else
    print_warning "FastQC verification failed"
fi

# Step 7: Create activation script
print_status "Creating activation script..."
cat > "$HOME/activate_seasnake.sh" << 'EOF'
#!/bin/zsh
# SEAsnake Environment Activation Script

# Activate conda
eval "$(~/miniforge3/bin/conda shell.zsh hook)"

# Activate SEAsnake environment
conda activate SEAsnake

echo "SEAsnake environment activated!"
echo "Your prompt should now show (SEAsnake)"
echo ""
echo "To deactivate when done, run: conda deactivate"
EOF

chmod +x "$HOME/activate_seasnake.sh"

# Final instructions
print_success "Setup completed successfully!"
echo ""
print_status "To use SEAsnake:"
echo "1. Run: source ~/activate_seasnake.sh"
echo "2. Or manually activate with: eval \"\$(~/miniforge3/bin/conda shell.zsh hook)\" && conda activate SEAsnake"
echo ""
print_status "SEAsnake directory: $SEASNAKE_DIR"
print_status "Activation script: $HOME/activate_seasnake.sh"
echo ""
print_warning "Remember: Always activate the SEAsnake environment before running bioinformatics tools!"