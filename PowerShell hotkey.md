# PowerShell hotkey

We have a different kind of an article today. Instead of talking about PowerShell commands or PowerShell scripting, we'll look at interacting with the Windows PowerShell *shell* itself, and how you can use keyboard shortcuts to speed up our work.

## Clear Screen: CTRL+L

I need to establish a clean screen before I do anything serious in PowerShell. Pressing CTRL+L is faster than running Clear-Host (and it works in Linux shells too!)

![img](https://images.contentstack.io/v3/assets/blt36c2e63521272fdc/blt906d9d5231997ca4/62d4b6290d8ac2360dc725bc/powershell-ctrl-l.gif)

## Faster Copy Paste

In Windows PowerShell, you can highlight a command with your mouse then press CTRL+C to copy. Paste with CTRL+C. No big revelation there.

But Windows PowerShell also supports automatic command highlighting with CTRL+A. Once it's highlighted, press CTRL+C. No mouse required!

![img](https://images.contentstack.io/v3/assets/blt36c2e63521272fdc/bltd93975cbf1953547/62d4b629d612b23647236d70/powershell-ctrl-a.gif)

## History Recall: CTRL+R

You can press the up arrow to go back through your PowerShell session history, but you can also search quickly. Press CTRL+R and type a few letters that you remember from the previous command, and PowerShell will show you any matches.

![img](https://images.contentstack.io/v3/assets/blt36c2e63521272fdc/blt7fa441918adc926c/62d4b62907121b35836b7bed/powershell-ctrl-r.gif)

## Interactive Editing: Shift+Enter

Let's say you copy-paste a PowerShell 1-liner from [some GitHub gist page](https://gist.github.com/joswr1ght/c557f8627832d54458c810e43be9c055). You can move the cursor using the left and right arrows and add a new line without executing the command by pressing Shift+Enter. Press Ctrl+Enter if you want to add a new line above the current line.

![img](https://images.contentstack.io/v3/assets/blt36c2e63521272fdc/blt5c92ba63bf69624d/62d4b62a429ac43379ca46a0/powershell-shift-enter.gif)

## Last Argument Copy: Alt+.

The last argument to a command is something that is often reused. You don't have to type it more than once: press Alt+. (Option + . on macOS) to insert the last argument of the previous command automatically.

![img](https://images.contentstack.io/v3/assets/blt36c2e63521272fdc/bltc70a615e74ae02d3/62d4b629b8a7f1380fcd4840/powershell-alt-dot.gif)

## Select Completion: Ctrl+Space

We know you can press Tab to complete a command or option, but you can also press Ctrl+Space to get a visual selection of matching commands, choosing the one you want with arrow keys. If help information is available, it will be displayed automatically.

![img](https://images.contentstack.io/v3/assets/blt36c2e63521272fdc/blt448417476237abc8/62d4b62a5648fd328b89f75b/powershell-ctrl-space.gif)

## Numeric Arguments: Alt+N

You can press Alt+Number (e.g., Alt+7) then any added character to input that many characters. If you need more than 9 characters, keep holding Alt and press the number of characters you need.

For example, want to enter 62 * characters? Press Alt+62, then press *:

![img](https://images.contentstack.io/v3/assets/blt36c2e63521272fdc/blt3aef92f4df4fd604/62d4b629181754349ea32f40/powershell-alt-62-asterisk.gif)

Want to backspace 7 times? Alt+7, then press backspace:

![img](https://images.contentstack.io/v3/assets/blt36c2e63521272fdc/blt4f18dbbdb30f7421/62d4b62927582a3696ad5865/powershell-alt-7-backspace.gif)

This works for arrow keys and any other repeated input characters as well.

## Command Dump: Ctrl+Alt+Shift+?

Finally, get a list of keyboard shortcuts by pressing Ctrl+Alt+Shift+?

![img](https://images.contentstack.io/v3/assets/blt36c2e63521272fdc/blta52aa24930ba31da/62d4b62b6dcb57349d32f194/powershell-ctrl-alt-shift-question.gif)

## Conclusion

Some of these I already use all the time, but others I probably won't use. But they are all useful anytime you feel like you need to +2 your PowerShell game.



-Joshua Wright