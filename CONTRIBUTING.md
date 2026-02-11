# Contributing to Pwnagotchi Pi Zero 2 W Guide

Thank you for your interest in contributing! 🎉

This guide is the result of systematic troubleshooting and comprehensive documentation. We welcome contributions that maintain this standard of quality.

---

## 🤝 How to Contribute

### Reporting Issues

**Before opening an issue:**
1. Search existing issues (including closed ones)
2. Try the troubleshooting guide
3. Test on a clean installation if possible

**When reporting bugs:**
- Use the issue template (if available)
- Include full error messages
- Provide system information (OS, kernel, hardware)
- Describe steps to reproduce
- Include what you've already tried

**Example good bug report:**
```
Title: Monitor mode not working after reboot

Hardware: Pi Zero 2 W
Kernel: 6.12.62+rpt-rpi-v8
OS: Debian Trixie

Description:
After rebooting the Pi, mon0 interface doesn't get created.
USB is disconnected. Nexmon appears to load (checked dmesg).

Steps to reproduce:
1. Fresh install following BUILD-GUIDE.md
2. Verify mon0 works initially
3. Reboot
4. mon0 not present

Already tried:
- Checking USB is disconnected
- Verifying Nexmon firmware is present
- Checking systemd logs

Logs:
[paste relevant logs here]
```

---

### Suggesting Enhancements

We welcome suggestions for:
- Improved documentation clarity
- Additional hardware compatibility
- New features or workflows
- Better troubleshooting steps

**Good enhancement requests include:**
- Clear description of the problem/need
- Proposed solution
- Why this would help others
- Any implementation details

---

### Contributing Code/Documentation

#### Documentation Improvements

**We especially welcome:**
- Clarifications for confusing steps
- Additional examples
- Better explanations
- Corrections for errors
- New hardware compatibility info
- Alternative methods that work

**Guidelines:**
- Keep the same friendly, clear tone
- Use emoji sparingly but appropriately
- Include code blocks with proper syntax highlighting
- Test all commands/procedures before submitting
- Maintain the "written for someone forgetful" approach

#### Code Contributions

**For scripts and utilities:**
- Follow existing code style
- Add comments explaining non-obvious parts
- Test thoroughly on Pi Zero 2 W
- Document any dependencies
- Include usage examples

#### Pull Request Process

1. **Fork the repository**

2. **Create a branch:**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes:**
   - Follow existing structure and style
   - Test everything thoroughly
   - Update relevant documentation

4. **Commit with clear messages:**
   ```bash
   git commit -m "Add: Clear description of what you changed"
   ```
   
   **Good commit messages:**
   - `Fix: Correct Nexmon build commands for kernel 6.12.65`
   - `Add: Hardware compatibility info for WaveShare 4" display`
   - `Improve: Clarify USB gadget conflict in BUILD-GUIDE.md`
   - `Docs: Fix typo in TROUBLESHOOTING.md`

5. **Push to your fork:**
   ```bash
   git push origin feature/your-feature-name
   ```

6. **Open a Pull Request**
   - Clear title describing the change
   - Reference any related issues
   - Describe what you changed and why
   - Include testing details

---

## 📝 Documentation Standards

### Writing Style

- **Clear and concise** - No unnecessary jargon
- **Step-by-step** - Assume reader is following along
- **Tested procedures** - Every command should be verified
- **Helpful warnings** - Use ⚠️ for critical information
- **Checkpoints** - Include ✅ validation steps

### Formatting

**Use consistent markdown:**

```markdown
### Section Heading

**Bold for emphasis** on important concepts

`Code formatting` for commands and filenames

```bash
# Code blocks with syntax highlighting
command --with-flags
```

- Bullet lists for options
- Numbered lists for sequences

> **Note:** Information boxes for tips

⚠️ **Warning:** Critical information
✅ **Success:** Validation checkpoints
```

**File paths:**
- Use absolute paths: `/etc/pwnagotchi/config.toml`
- Not: `config.toml`

**Commands:**
- Show full commands: `sudo systemctl restart pwnagotchi`
- Not: `restart the service`

---

## 🧪 Testing Requirements

**Before submitting:**

1. **Test on clean installation**
   - Fresh Raspberry Pi OS Lite
   - Follow your modified procedure
   - Document any issues

2. **Verify all commands work**
   - No typos
   - Correct syntax
   - Expected output matches

3. **Check documentation renders correctly**
   - Preview markdown locally
   - Verify links work
   - Check formatting

4. **Test edge cases**
   - What if command fails?
   - What if file doesn't exist?
   - Include error handling

---

## 🎯 Priority Areas

We especially need help with:

1. **Hardware Testing**
   - Testing on different Pi models (Pi 3/4/5)
   - Different display compatibility
   - Other WiFi chipsets

2. **Documentation Improvements**
   - Simpler explanations
   - More examples
   - Better troubleshooting

3. **Alternative Methods**
   - Different build approaches
   - Performance optimizations
   - Plugin examples

4. **Internationalization**
   - Translations
   - Region-specific notes

---

## ❌ What We Don't Accept

- Incomplete or untested changes
- Breaking existing functionality
- Removing safety warnings
- Encouraging illegal activity
- Poorly documented code
- Plagiarized content

---

## 📜 Code of Conduct

### Our Standards

- **Be respectful** - Different skill levels, be patient
- **Be constructive** - Criticism should be helpful
- **Be collaborative** - Work together to solve problems
- **Be inclusive** - Welcome all contributors
- **Assume good faith** - People are trying to help

### Unacceptable Behavior

- Harassment or discriminatory language
- Personal attacks
- Trolling or deliberately unhelpful responses
- Spam or off-topic discussions
- Sharing others' private information

---

## 📞 Questions?

- **Documentation questions:** Open a Discussion
- **Bug reports:** Open an Issue
- **Feature requests:** Open an Issue with [FEATURE] tag
- **Security issues:** Email privately (don't open public issue!)

---

## 🏆 Recognition

Contributors will be:
- Listed in a CONTRIBUTORS.md file
- Credited in relevant documentation
- Thanked in release notes

We value all contributions, big or small! 💪

---

## 📚 Resources

- [BUILD-GUIDE.md](docs/BUILD-GUIDE.md) - Full build procedure
- [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) - Known issues
- [QUICK-REFERENCE.md](docs/QUICK-REFERENCE.md) - Command reference

---

<p align="center">
  <strong>Thank you for helping make this guide better! 🎉</strong><br>
  <em>Every contribution helps someone build their Pwnagotchi successfully!</em>
</p>
