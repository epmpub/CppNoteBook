# upgrade gcc-14

### On Debian/Ubuntu-based systems using update-alternatives:

```bash
# First make sure you have g++-14 installed
sudo apt-get install g++-14

# Then set it as an alternative
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-14 140
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-14 140

# Make it the default choice
sudo update-alternatives --set g++ /usr/bin/g++-14
sudo update-alternatives --set gcc /usr/bin/gcc-14
```

### Using symbolic links:

```bash
# Remove old links if they exist
sudo rm -f /usr/bin/g++ /usr/bin/gcc

# Create new symbolic links
sudo ln -s /usr/bin/g++-14 /usr/bin/g++
sudo ln -s /usr/bin/gcc-14 /usr/bin/gcc
```

### Using shell aliases:

Add these lines to your `~/.bashrc` or `~/.zshrc`:

```bash
alias g++='g++-14'
alias gcc='gcc-14'
```

Then run `source ~/.bashrc` or `source ~/.zshrc` to apply changes.

### Verify the change:

After applying one of these methods, verify that g++-14 is now your default compiler:

```bash
g++ --version
```

This should show that you're using g++ version 14.x.x.