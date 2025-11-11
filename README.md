# 🎨 Proxmox Catppuccin Theme

Transform your Proxmox VE web interface with the soothing Catppuccin color palette. This refresh ships Latte (light) and Macchiato (dark) variants that react to both your Proxmox preference and your OS color scheme.

> **Inspired by** [SolarPVE](https://github.com/dabeastnet/SolarPVE) - A Solarized-inspired theme for Proxmox VE
## 📸 Preview

### Light Theme (Catppuccin Latte)
![Light Theme - Catppuccin Latte](./Images/light-catppuccin.png)

### Dark Theme (Catppuccin Mocha)
![Dark Theme - Catppuccin Mocha](./Images/dark-catppuccin.png)

## 🎨 Custom Logos

### Dark Theme Logo
![Dark Logo](./Images/logo-128-dark.png)

### Light Theme Logo  
![Light Logo](./Images/logo-128-light.png)

### Upload Custom Logo via SCP

To upload your custom logo to the Proxmox server:

```bash
# Upload dark theme logo
scp /path/to/your/logo-128-dark.png root@YOUR_SERVER_IP:/usr/share/pve-manager/images/

# Upload light theme logo  
scp /path/to/your/logo-128-light.png root@YOUR_SERVER_IP:/usr/share/pve-manager/images/

# Example (Windows PowerShell):
PS C:\Users\YOU\Desktop> scp C:\Users\YOU\Desktop\logo-128-dark.png root@192.253.1.103:/usr/share/pve-manager/images/

```

After uploading, restart the Proxmox service:
```bash
sudo systemctl restart pveproxy
```

### Troubleshooting Logo Issues

If the custom logo doesn't appear:

1. **Check file permissions:**
   ```bash
   sudo chmod 644 /usr/share/pve-manager/images/logo-128-*.png
   ```

2. **Clear browser cache** (Ctrl+F5 or Ctrl+Shift+R)

3. **Check browser developer tools** (F12) for any CSS conflicts

4. **Verify file exists:**
   ```bash
   ls -la /usr/share/pve-manager/images/logo-128-*.png
   ```

5. **Alternative: Replace default logo directly:**
   ```bash
   sudo cp /usr/share/pve-manager/images/logo-128-dark.png /usr/share/pve-manager/images/proxmox_logo.png
   ```
## ✨ What's Included

- **Adaptive theme switching** – Works with manual, auto, and OS-driven dark mode
- **Polished Catppuccin surfaces** – Panels, navigation, console, login, & scrollbars
- **Semantic color tokens** – Quick theming via friendly CSS custom properties
- **Dependency-free helper** – Tiny vanilla script keeps classes in sync
- **Drop-in install** – Copy one CSS file and one snippet into the default template

## 🚀 Quick Start

### Step 1: Deploy the CSS
```bash
# Copy the theme file to Proxmox's static assets
sudo cp catppuccin.css /usr/share/pve-manager/images/
```

### Step 2: Update the Template
Modify `/usr/share/pve-manager/index.html.tpl` by dropping the helper snippet **just before** `</head>` (or copy it directly from this repo):

```html
<script>
  document.addEventListener('DOMContentLoaded', () => {
    const DARK_THEME_SELECTOR = 'link[href*="theme-proxmox-dark.css"]';
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)');

    const isDarkLinkEl = (node) =>
      node &&
      node.tagName === 'LINK' &&
      typeof node.href === 'string' &&
      node.href.includes('theme-proxmox-dark.css');

    const isDarkModeActive = () => {
      const darkLink = document.querySelector(DARK_THEME_SELECTOR);
      if (!darkLink) {
        return false;
      }

      const { media } = darkLink;
      if (!media || media === 'all') {
        return true;
      }

      try {
        return window.matchMedia(media).matches;
      } catch (error) {
        return prefersDark.matches;
      }
    };

    const applyThemeClass = () => {
      const darkMode = isDarkModeActive();
      document.body.classList.toggle('proxmox-theme-dark', darkMode);
      document.body.classList.toggle('proxmox-theme-light', !darkMode);
    };

    applyThemeClass();

    if (typeof prefersDark.addEventListener === 'function') {
      prefersDark.addEventListener('change', applyThemeClass);
    } else if (typeof prefersDark.addListener === 'function') {
      prefersDark.addListener(applyThemeClass);
    }

    const observer = new MutationObserver((mutations) => {
      const relevant = mutations.some((mutation) => {
        if (mutation.type === 'attributes' && isDarkLinkEl(mutation.target)) {
          return true;
        }

        const nodes = [
          ...mutation.addedNodes,
          ...mutation.removedNodes,
        ];
        return nodes.some((node) => isDarkLinkEl(node));
      });

      if (relevant) {
        applyThemeClass();
      }
    });

    observer.observe(document.head, {
      childList: true,
      subtree: true,
      attributes: true,
      attributeFilter: ['media', 'href'],
    });
  });
</script>
<link rel="stylesheet" href="/pve2/images/catppuccin.css">
```

### Step 3: Restart Services
```bash
sudo systemctl restart pveproxy
```

### Step 4: Clear Cache
Hard refresh your browser (Ctrl+F5) to see the changes.

## 🎯 How It Works

The CSS exports semantic tokens (backgrounds, borders, accents) that the helper script swaps between Latte and Macchiato:

- When the Proxmox dark bundle is active, the script toggles `proxmox-theme-dark` and Macchiato tokens take over
- When it is absent, the body receives `proxmox-theme-light` and Latte tokens are used
- Auto/OS dark mode is handled via `matchMedia` listeners and a `MutationObserver`, so hot theme changes stay in sync without reloading

## 🎨 Customization

Tweak the semantic tokens in `catppuccin.css` to retune the vibe without hunting every selector.

```css
:root {
  --pve-focus: var(--ctp-teal);      /* Accent for buttons, focus rings */
  --pve-success: var(--ctp-green);   /* VM running state */
  --pve-warning: var(--ctp-yellow);  /* Pending/paused */
  --pve-danger: var(--ctp-red);      /* Errors & failures */
  --pve-scrollbar-thumb: #d2d7e0;    /* Optional lighter scrollbars */
  /* …see the file for more tokens */
}

body.proxmox-theme-dark {
  --pve-focus: var(--ctp-sapphire);  /* Swap dark-mode accent */
  --pve-scrollbar-thumb: #575c76;
}
```

- **Want a different accent?** Override `--pve-focus` in both blocks.
- **Prefer flat panels?** Set `--pve-shadow: none;`.
- **Disable login glow?** Remove the `#loginform` block at the end.

## 🖥️ Terminal Customization

### Proxmox Shell Theme

For a complete Catppuccin experience, customize your Proxmox shell environment:

### Quick Setup (Recommended)

Add this to your `~/.bashrc` or `~/.zshrc`:

```bash
# ✅ Catppuccin Macchiato background/text (лише для інтерактивного shell)
if [[ $- == *i* ]]; then
    printf '\033]11;#181926\007'  # background: crust
    printf '\033]10;#cad3f5\007'  # foreground: text
    echo -ne ''
fi

# ✅ Enable colored output for commands
export CLICOLOR=1
export LSCOLORS=ExFxBxDxCxegedabagacad

# Aliases for colored `ls` and safer commands
alias ls='ls --color=auto'
alias ll='ls -alF --color=auto'
alias la='ls -A --color=auto'
alias l='ls -CF --color=auto'

alias grep='grep --color=auto'
alias fgrep='fgrep --color=auto'
alias egrep='egrep --color=auto'

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# ✅ Cursor: green & blinking (Catppuccin accent green)
if [[ $- == *i* ]]; then
    echo -ne '\e]12;#a6da95\a'     # cursor color
    echo -ne '\e[?12;25h'          # blinking on
fi

# ✅ Prompt: green username@host, cyan path, gray git branch
if [ -x "$(command -v git)" ]; then
    parse_git_branch() {
        git branch 2>/dev/null | sed -n '/\* /s///p'
    }
    PS1='\[\e[38;5;110m\]\u@\h \[\e[38;5;109m\]\W\[\e[38;5;247m\]$(parse_git_branch)\[\e[0m\] \$ '
else
    PS1='\[\e[38;5;110m\]\u@\h \[\e[38;5;109m\]\W\[\e[0m\] \$ '
fi
```

### Apply Changes
```bash
# Reload shell configuration
source ~/.bashrc  # or source ~/.zshrc

# Or restart your terminal session
```




## 🤝 Contributing

Found a bug or want to improve something?

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-thing`)
3. Commit your changes (`git commit -m 'Add amazing thing'`)
4. Push to the branch (`git push origin feature/amazing-thing`)
5. Open a Pull Request

## 📄 License

This project is licensed under **CC BY-NC 4.0**.

- ✅ Free for non-commercial use with attribution
- ❌ Commercial use requires separate licensing

---

## 📋 TODO

- [x] Add custom Catppuccin-themed logo (`logo-128-dark.png`)
- [x] Create light variant logo (`logo-128-light.png`)
- [ ] Update favicon references in template

---

**Enjoy your beautifully themed Proxmox interface!** 🌟
