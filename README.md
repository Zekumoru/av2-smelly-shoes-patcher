# Smelly Shoes RXDATA Save Patcher

A small browser-based save patcher that adds **Smelly Shoes** to an Aveyond 2 (Version 2) RXDATA save file.

Smelly Shoes is Armor **#84** internally. It is an equippable armor item for Ean that acts like a permanent beast repellent, preventing monsters from appearing on the map.

The patcher runs entirely in the browser. It does **not** upload saves anywhere.

## How to use

1. **Back up your save file first.**
   - Copy your original `SaveX.rxdata` somewhere safe before patching.
2. Open the patcher page in your browser.
3. Drag your `SaveX.rxdata` file onto the upload box.
   - You can also click the box and choose the file manually.
4. Click **Patch save with Smelly Shoes**.
5. Your browser will download the patched save using the same filename.
6. Replace your original save with the patched one.
   - Depending on your browser/download settings, the file may land in your Downloads folder first.
   - If your browser renames it to something like `Save6 (1).rxdata`, rename it back to the original save slot name, for example `Save6.rxdata`.

## What it changes

The patcher only changes the party armor inventory.

It adds or updates this value:

```ruby
@armors[84] = 1
```

That means:

```text
Armor #84 = 1
```

It does **not** change:

- story flags
- switches
- variables
- event choices
- characters
- equipped gear
- weapons
- normal items
- money
- steps
- playtime

If Smelly Shoes is already present, the patcher leaves it alone as long as the count is at least `1`.

## How it works

RPG Maker `.rxdata` save files, which are used in Aveyond 2, are Ruby Marshal binary files. Ruby normally reads them with something like:

```ruby
Marshal.load(File.binread("Save6.rxdata"))
```

This patcher does not need Ruby because it does not fully decode or rebuild the whole save file. Instead, it performs a small targeted binary patch directly in JavaScript.

The browser script does this:

1. Reads the selected save with the browser File API.
2. Checks that the file looks like a Ruby Marshal file.
   - Ruby Marshal files usually begin with the bytes `04 08`.
3. Searches the binary data for the inventory field named:

```text
@armors
```

1. Confirms that the value after `@armors` is a Ruby Marshal hash.
   - Ruby Marshal uses byte `0x7B` for a hash.
2. Reads the hash length and walks through the armor inventory entries.
3. Looks for armor ID `84`.
4. If armor `84` exists but has a count lower than `1`, it updates the count to `1`.
5. If armor `84` does not exist, it inserts a new hash entry for:

```text
84 => 1
```

1. Downloads the patched bytes as a new `.rxdata` file.

So the patcher is not a general save editor. It only understands enough of Ruby Marshal to safely find and patch the `@armors` inventory hash.

## Why no Ruby or Python is needed

Ruby and Python were useful for researching the save format, but players do not need either of them.

The final patcher is just a static HTML file with JavaScript inside. Every modern browser can run JavaScript, read a local file selected by the user, edit the bytes in memory, and download the patched file.

## Safety notes

- Always back up your save before patching.
- The patcher is made specifically for this Aveyond 2 (Version 2) RXDATA save structure.
- [SPOILER] It does not bypass the event choice: choosing to blow away the Troll or give the North Wind back to wind lab; it only adds the resulting armor item to the inventory.
- Saves are processed locally in the browser and are not uploaded to a server.
- The page uses no external libraries.
