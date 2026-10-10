# Terminal
instalador de herramientas para terminal.


#!/bin/bash
set -e

# ──────────────────────────────────────────────
# Colores
# ──────────────────────────────────────────────
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m'

info()  { echo -e "${GREEN}[✓]${NC} $1"; }
warn()  { echo -e "${YELLOW}[!]${NC} $1"; }
error() { echo -e "${RED}[✗]${NC} $1"; exit 1; }

echo "=========================================="
echo "  Terminal Setup - macOS"
echo "=========================================="
echo ""

# ──────────────────────────────────────────────
# 1. Homebrew
# ──────────────────────────────────────────────
if ! command -v brew &>/dev/null; then
    warn "Instalando Homebrew..."
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    if [[ $(uname -m) == "arm64" ]]; then
        echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
        eval "$(/opt/homebrew/bin/brew shellenv)"
    else
        echo 'eval "$(/usr/local/bin/brew shellenv)"' >> ~/.zprofile
        eval "$(/usr/local/bin/brew shellenv)"
    fi
    info "Homebrew instalado."
else
    info "Homebrew ya está instalado."
fi

BREW_PREFIX=$(brew --prefix)

# ──────────────────────────────────────────────
# 2. Oh My Zsh
# ──────────────────────────────────────────────
if [ ! -d "$HOME/.oh-my-zsh" ]; then
    warn "Instalando Oh My Zsh..."
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
    info "Oh My Zsh instalado."
else
    info "Oh My Zsh ya está instalado."
fi

# ──────────────────────────────────────────────
# 3. Herramientas
# ──────────────────────────────────────────────
info "Instalando herramientas (esto puede tardar)..."
brew install zsh-autosuggestions zsh-syntax-highlighting zsh-completions \
             zsh-history-substring-search fzf zoxide ripgrep lsd bat
info "Herramientas instaladas."

# ──────────────────────────────────────────────
# 4. Nerd Font
# ──────────────────────────────────────────────
if ! ls ~/Library/Fonts/ 2>/dev/null | grep -qi "nerd"; then
    warn "Instalando JetBrainsMono Nerd Font..."
    brew install --cask font-jetbrains-mono-nerd-font
    info "Nerd Font instalada."
    echo "   ⚠️  Configúrala manualmente en tu terminal:"
    echo "   iTerm2 → Preferences → Profiles → Text → Font: JetBrainsMono Nerd Font"
    echo "   Terminal.app → Ajustes → Perfil → Fuente: JetBrainsMono Nerd Font"
else
    info "Nerd Font ya está instalada."
fi

# ──────────────────────────────────────────────
# 5. Configurar .zshrc
# ──────────────────────────────────────────────
ZSHRC="$HOME/.zshrc"
BACKUP="${ZSHRC}.bak.$(date +%Y%m%d%H%M%S)"
cp "$ZSHRC" "$BACKUP"
info "Backup creado: $BACKUP"

# Plugins de Oh My Zsh (mantener solo git, el resto va manual)
if grep -q "plugins=(" "$ZSHRC"; then
    sed -i '' 's/plugins=(.*/plugins=(git)/' "$ZSHRC"
else
    echo 'plugins=(git)' >> "$ZSHRC"
fi

# Bloque idempotente: se reemplaza en cada ejecución
MARKER_BEGIN="# >>> terminal-setup >>>"
MARKER_END="# <<< terminal-setup <<<"

if grep -qF "$MARKER_BEGIN" "$ZSHRC"; then
    sed -i '' "/${MARKER_BEGIN// /\\ }/,/${MARKER_END// /\\ }/d" "$ZSHRC"
fi

cat >> "$ZSHRC" << EOF

$MARKER_BEGIN
# Completions extendidos
fpath=($BREW_PREFIX/share/zsh-completions \$fpath)
autoload -Uz compinit && compinit

# fzf (Ctrl+R, Ctrl+T, Ctrl+G)
eval "\$(fzf --zsh)"

# zsh-autosuggestions (flecha →)
source $BREW_PREFIX/share/zsh-autosuggestions/zsh-autosuggestions.zsh

# zsh-history-substring-search (↑↓)
source $BREW_PREFIX/share/zsh-history-substring-search/zsh-history-substring-search.zsh
bindkey '^[[A' history-substring-search-up
bindkey '^[[B' history-substring-search-down

# zsh-syntax-highlighting (SIEMPRE al final de los plugins)
source $BREW_PREFIX/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

# zoxide (después de compinit)
eval "\$(zoxide init zsh)"

# Aliases
alias ls='lsd'
alias ll='lsd -l'
alias la='lsd -la'
alias lt='lsd --tree'
alias cat='bat'
alias catn='/bin/cat'
alias grep='grep --color=auto'
$MARKER_END
EOF

info ".zshrc actualizado."

# ──────────────────────────────────────────────
# 6. Resumen
# ──────────────────────────────────────────────
echo ""
echo "=========================================="
info "¡Todo listo! Abre una nueva terminal para aplicar."
echo "=========================================="
echo ""
echo "  Atajos principales:"
echo "  ┌─────────────────────────────────────────────────────┐"
echo "  │ Ctrl+R          → buscar en historial (fzf)        │"
echo "  │ Ctrl+T          → buscar archivo (fzf)             │"
echo "  │ ↑ / ↓           → buscar historial por substring   │"
echo "  │ → (flecha der.) → aceptar autosuggestion           │"
echo "  │ Tab             → completion (zsh-completions)     │"
echo "  │ z <nombre>      → saltar a directorio (zoxide)     │"
echo "  │ zi <nombre>     → zoxide + fzf (interactivo)       │"
echo "  │ ls / ll / la    → lsd (iconos + colores)           │"
echo "  │ lt              → lsd en árbol                     │"
echo "  │ cat <archivo>   → bat (syntax highlighting)        │"
echo "  │ catn <archivo>  → cat original                     │"
echo "  └─────────────────────────────────────────────────────┘"
echo ""
echo "  Si algo falla, restaura el backup:"
echo "  cp $BACKUP ~/.zshrc"
echo ""   
