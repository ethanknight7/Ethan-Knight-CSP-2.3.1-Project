2.3.1 encryption and decryption ##############################################################ENCODE################################################################## """
VISUAL ENCODER — encode.py PLTW Computer Science | Unit 2 Activity 2.3.1
Uses ONLY built-in Python libraries. No pip install required.

HOW TO RUN: python encode.py

OUTPUT: encoded_message.png saved in the same folder
"""

import turtle import random import struct import zlib import tkinter as tk from tkinter import font as tkfont

── CONSTANTS ────────────────────────────────────────────────
STAMP_SIZE = 20 # size of each encoded character square GRID_COLS = 20 # characters per row START_X = -390 # turtle x of the first message stamp START_Y = 290 # turtle y of the first message stamp NOISE_COUNT = 400 # number of background noise shapes CANVAS_W = 800 # turtle window width CANVAS_H = 600 # turtle window height OUTPUT_FILE = "encoded_message.png"

── VISUAL THEME ─────────────────────────────────────────────
BG_COLOR = "#0a0e1a" # deep navy background BORDER_COLOR = "#00e5ff" # cyan accent for the frame GRID_OUTLINE = "#1a2a3a" # subtle dark teal grid panel tint PANEL_COLOR = "#0d1f2d" # slightly lighter panel behind message grid

── FUNCTION: styled input dialog ────────────────────────────
def get_message_from_user(): """ Opens a custom-styled Tkinter window (dark theme, cyan accents) for the user to type their secret message. Returns the entered string, or 'HELLO WORLD' if canceled. """ result = {"value": None}

# Build a styled top-level window
root = tk.Tk()
root.title("Visual Encoder")
root.configure(bg="#0a0e1a")
root.resizable(False, False)

# Center the window on screen
win_w, win_h = 460, 220
sx = root.winfo_screenwidth()
sy = root.winfo_screenheight()
root.geometry("{}x{}+{}+{}".format(win_w, win_h,
              (sx - win_w) // 2, (sy - win_h) // 2))

# Title label
title_lbl = tk.Label(root,
    text      = "[ VISUAL ENCODER ]",
    bg        = "#0a0e1a",
    fg        = "#00e5ff",
    font      = ("Courier New", 15, "bold"),
    pady      = 12
)
title_lbl.pack()

# Subtitle / prompt
prompt_lbl = tk.Label(root,
    text = "Enter your secret message below:",
    bg   = "#0a0e1a",
    fg   = "#8899aa",
    font = ("Courier New", 10)
)
prompt_lbl.pack()

# Entry field with dark background
entry_var = tk.StringVar()
entry = tk.Entry(root,
    textvariable = entry_var,
    bg           = "#111c2b",
    fg           = "#e0f7ff",
    insertbackground = "#00e5ff",
    font         = ("Courier New", 12),
    relief       = tk.FLAT,
    width        = 34,
    bd           = 0,
    highlightthickness = 2,
    highlightcolor     = "#00e5ff",
    highlightbackground = "#1a2a3a"
)
entry.pack(pady=10, ipady=6)
entry.focus()

# Button row
btn_frame = tk.Frame(root, bg="#0a0e1a")
btn_frame.pack(pady=6)

def on_encode():
    result["value"] = entry_var.get().strip() or "HELLO WORLD"
    root.destroy()

def on_cancel():
    result["value"] = "HELLO WORLD"
    root.destroy()

encode_btn = tk.Button(btn_frame,
    text    = "  ENCODE  ",
    command = on_encode,
    bg      = "#00e5ff",
    fg      = "#0a0e1a",
    font    = ("Courier New", 10, "bold"),
    relief  = tk.FLAT,
    cursor  = "hand2",
    padx    = 10, pady = 4
)
encode_btn.pack(side=tk.LEFT, padx=8)

cancel_btn = tk.Button(btn_frame,
    text    = "  CANCEL  ",
    command = on_cancel,
    bg      = "#1a2a3a",
    fg      = "#8899aa",
    font    = ("Courier New", 10),
    relief  = tk.FLAT,
    cursor  = "hand2",
    padx    = 10, pady = 4
)
cancel_btn.pack(side=tk.LEFT, padx=8)

# Allow Enter key to submit
root.bind("<Return>", lambda e: on_encode())

root.mainloop()
return result["value"]
── FUNCTION: styled completion dialog ───────────────────────
def show_success_dialog(filepath): """ Shows a styled dark-theme dialog confirming the PNG was saved. """ root = tk.Tk() root.title("Encoding Complete") root.configure(bg="#0a0e1a") root.resizable(False, False)

win_w, win_h = 420, 240
sx = root.winfo_screenwidth()
sy = root.winfo_screenheight()
root.geometry("{}x{}+{}+{}".format(win_w, win_h,
              (sx - win_w) // 2, (sy - win_h) // 2))

tk.Label(root,
    text  = "[ ENCODING COMPLETE ]",
    bg    = "#0a0e1a",
    fg    = "#00e5ff",
    font  = ("Courier New", 13, "bold"),
    pady  = 14
).pack()

tk.Label(root,
    text = "File saved:  " + filepath,
    bg   = "#0a0e1a",
    fg   = "#e0f7ff",
    font = ("Courier New", 10)
).pack()

tk.Label(root,
    text = "\nSend the receiver:\n  1.  " + filepath + "\n  2.  decode.py\n\nUse SEPARATE channels for security.",
    bg   = "#0a0e1a",
    fg   = "#8899aa",
    font = ("Courier New", 9),
    justify = tk.LEFT
).pack(padx=20)

tk.Button(root,
    text    = "  OK  ",
    command = root.destroy,
    bg      = "#00e5ff",
    fg      = "#0a0e1a",
    font    = ("Courier New", 10, "bold"),
    relief  = tk.FLAT,
    cursor  = "hand2",
    padx=14, pady=4
).pack(pady=12)

root.mainloop()
── FUNCTION: convert a character to an RGB color ────────────
def ascii_to_rgb(char): """ Encodes a character as (r, g, b). Red channel stores the ASCII value: r = ASCII_value * 2 Decoder reverses with: ASCII_value = r // 2 """ code = ord(char) r = min(code * 2, 255) g = (code * 3) % 256 b = (code * 7) % 256 return (r, g, b)

── FUNCTION: format RGB as a hex string ─────────────────────
def rgb_to_hex(r, g, b): return "#{:02x}{:02x}{:02x}".format(r, g, b)

── FUNCTION: map character index to turtle coordinates ──────
def grid_position(index): """ Returns (x, y) turtle coordinates for character at index. Fills left-to-right, wraps to next row every GRID_COLS chars. """ col = index % GRID_COLS row = index // GRID_COLS x = START_X + col * STAMP_SIZE y = START_Y - row * STAMP_SIZE return (x, y)

── FUNCTION: stamp one encoded character square ─────────────
def stamp_character(t, char, index): """ Moves turtle to grid position and stamps a colored square encoding the character's ASCII value in the red channel. """ x, y = grid_position(index) r, g, b = ascii_to_rgb(char) t.penup() t.goto(x, y) t.color(rgb_to_hex(r, g, b)) t.stamp()

── FUNCTION: draw the decorative canvas background ──────────
def draw_background(t, screen): """ Draws the visual theme onto the turtle canvas: - Deep navy fill (set via bgcolor) - A subtle darker panel behind where the message grid sits - A glowing cyan border frame around the whole canvas - Thin grid lines suggesting a technical / data aesthetic """ # Draw the dark panel behind the message grid area t.penup() t.goto(START_X - 10, START_Y + 10) t.pendown() t.color(PANEL_COLOR) t.begin_fill() panel_w = GRID_COLS * STAMP_SIZE + 20 panel_h = 6 * STAMP_SIZE + 20 # show ~6 rows depth for _ in range(2): t.forward(panel_w) t.right(90) t.forward(panel_h) t.right(90) t.end_fill() t.penup()

# Draw cyan border frame (inset slightly from edge)
margin = 12
t.goto(-CANVAS_W // 2 + margin, CANVAS_H // 2 - margin)
t.pendown()
t.color(BORDER_COLOR)
t.pensize(2)
for side in [CANVAS_W - margin*2, CANVAS_H - margin*2,
             CANVAS_W - margin*2, CANVAS_H - margin*2]:
    t.forward(side)
    t.right(90)
t.penup()
t.pensize(1)

# Corner accent marks — small ticks at each corner
corners = [
    (-CANVAS_W//2 + margin,  CANVAS_H//2 - margin, 0),
    ( CANVAS_W//2 - margin,  CANVAS_H//2 - margin, -90),
    ( CANVAS_W//2 - margin, -CANVAS_H//2 + margin, 180),
    (-CANVAS_W//2 + margin, -CANVAS_H//2 + margin,  90),
]
for (cx, cy, heading) in corners:
    t.goto(cx, cy)
    t.setheading(heading)
    t.pendown()
    t.color("#00ffcc")
    t.pensize(3)
    t.forward(18)
    t.penup()
t.pensize(1)

screen.update()
── FUNCTION: draw varied noise shapes for camouflage ────────
def draw_noise_from_list(t, noise_stamps, screen): """ Draws the pre-generated noise stamps onto the turtle canvas. Mixes square and circle shapes, and varies sizes slightly, making the image look like rich abstract digital art. """ for (nx, ny, nr, ng, nb, shape, size) in noise_stamps: t.penup() t.goto(nx, ny) t.color(rgb_to_hex(nr, ng, nb)) if shape == "circle": # Draw a filled circle using pendown arc trick t.setheading(0) t.pendown() t.begin_fill() t.circle(size, steps=12) t.end_fill() t.penup() else: # Square stamp — scale turtle shape to desired size t.shape("square") t.shapesize(size / 20) t.stamp() # Reset turtle shape back to square at STAMP_SIZE for message stamps t.shape("square") t.shapesize(STAMP_SIZE / 20) screen.update()

── FUNCTION: generate noise stamp data ──────────────────────
def generate_noise(): """ Pre-generates all noise stamp data as a list of tuples: (x, y, r, g, b, shape, size)

Mixing circles and squares, and varying sizes (8-22px),
makes the output image look more like abstract art and less
like an obvious grid of uniform squares.
"""
stamps = []
for _ in range(NOISE_COUNT):
    x     = random.randint(-CANVAS_W // 2, CANVAS_W // 2)
    y     = random.randint(-CANVAS_H // 2, CANVAS_H // 2)
    r     = random.randint(0, 255)
    g     = random.randint(0, 255)
    b     = random.randint(0, 255)
    shape = random.choice(["square", "square", "circle"])  # 2:1 squares:circles
    size  = random.choice([8, 10, 12, 14, 16, 20])
    stamps.append((x, y, r, g, b, shape, size))
return stamps
── FUNCTION: build the PNG pixel array from stamp data ──────
def build_pixel_array(message, noise_stamps): """ Builds a 2D pixel array matching exactly what was drawn on the turtle canvas, computed from the stamp data directly. pixels[y][x] = (r, g, b) """ # Start with the navy background color bg_r, bg_g, bg_b = 10, 14, 26 # matches BG_COLOR "#0a0e1a" pixels = [[(bg_r, bg_g, bg_b)] * CANVAS_W for _ in range(CANVAS_H)]

def fill_square(turtle_x, turtle_y, r, g, b, size=STAMP_SIZE):
    """Fills a size×size block centered at turtle coordinates."""
    cx = int(CANVAS_W / 2 + turtle_x)
    cy = int(CANVAS_H / 2 - turtle_y)
    half = size // 2
    for dy in range(-half, half):
        for dx in range(-half, half):
            px, py = cx + dx, cy + dy
            if 0 <= px < CANVAS_W and 0 <= py < CANVAS_H:
                pixels[py][px] = (r, g, b)

def fill_circle(turtle_x, turtle_y, r, g, b, radius):
    """Fills a circle centered at turtle coordinates."""
    cx = int(CANVAS_W / 2 + turtle_x)
    cy = int(CANVAS_H / 2 - turtle_y)
    for dy in range(-radius, radius + 1):
        for dx in range(-radius, radius + 1):
            if dx*dx + dy*dy <= radius*radius:
                px, py = cx + dx, cy + dy
                if 0 <= px < CANVAS_W and 0 <= py < CANVAS_H:
                    pixels[py][px] = (r, g, b)

# Draw panel background
panel_r, panel_g, panel_b = 13, 31, 45   # matches PANEL_COLOR "#0d1f2d"
panel_x = int(CANVAS_W / 2 + START_X - 10)
panel_y = int(CANVAS_H / 2 - (START_Y + 10))
panel_w = GRID_COLS * STAMP_SIZE + 20
panel_h = 6 * STAMP_SIZE + 20
for py in range(panel_y, min(panel_y + panel_h, CANVAS_H)):
    for px in range(panel_x, min(panel_x + panel_w, CANVAS_W)):
        if 0 <= px < CANVAS_W and 0 <= py < CANVAS_H:
            pixels[py][px] = (panel_r, panel_g, panel_b)

# Draw noise (squares and circles)
for (nx, ny, nr, ng, nb, shape, size) in noise_stamps:
    if shape == "circle":
        fill_circle(nx, ny, nr, ng, nb, size)
    else:
        fill_square(nx, ny, nr, ng, nb, size)

# Draw message character squares on top (always STAMP_SIZE)
for index, char in enumerate(message):
    x, y    = grid_position(index)
    r, g, b = ascii_to_rgb(char)
    fill_square(x, y, r, g, b, STAMP_SIZE)

# Draw end-of-message marker (black square)
end_x, end_y = grid_position(len(message))
fill_square(end_x, end_y, 0, 0, 0, STAMP_SIZE)

# Draw border frame (2px cyan line)
m = 12
border_r, border_g, border_b = 0, 229, 255   # "#00e5ff"
for px in range(m, CANVAS_W - m):
    for t in range(2):
        if 0 <= m+t < CANVAS_H: pixels[m+t][px] = (border_r, border_g, border_b)
        if 0 <= CANVAS_H-m-1-t < CANVAS_H: pixels[CANVAS_H-m-1-t][px] = (border_r, border_g, border_b)
for py in range(m, CANVAS_H - m):
    for t in range(2):
        if 0 <= m+t < CANVAS_W: pixels[py][m+t] = (border_r, border_g, border_b)
        if 0 <= CANVAS_W-m-1-t < CANVAS_W: pixels[py][CANVAS_W-m-1-t] = (border_r, border_g, border_b)

return pixels
── FUNCTION: write a PNG file using only built-in Python ────
def write_png(filepath, pixels): """ Saves the pixel array to a PNG file using struct + zlib only. No Pillow or external libraries needed. """ height = len(pixels) width = len(pixels[0])

def make_chunk(chunk_type, data):
    body = chunk_type + data
    crc  = zlib.crc32(body) & 0xFFFFFFFF
    return struct.pack(">I", len(data)) + body + struct.pack(">I", crc)

signature = b'\x89PNG\r\n\x1a\n'
ihdr_data = struct.pack(">IIBBBBB", width, height, 8, 2, 0, 0, 0)
ihdr      = make_chunk(b"IHDR", ihdr_data)

raw_rows = b""
for row in pixels:
    raw_rows += b'\x00'
    for (r, g, b) in row:
        raw_rows += bytes([r, g, b])

compressed = zlib.compress(raw_rows, level=6)
idat       = make_chunk(b"IDAT", compressed)
iend       = make_chunk(b"IEND", b"")

with open(filepath, "wb") as f:
    f.write(signature + ihdr + idat + iend)

print("[Encoder] Saved: " + filepath)
── MAIN PROGRAM ─────────────────────────────────────────────
def main():

# Step 1: get the secret message via styled dialog
message = get_message_from_user()
print("[Encoder] Encoding: '" + message + "'")

# Step 2: pre-generate noise stamp data
print("[Encoder] Generating noise…")
noise_stamps = generate_noise()

# Step 3: set up the turtle window with navy background
screen = turtle.Screen()
screen.title("Visual Encoder  //  encoding…")
screen.bgcolor(BG_COLOR)
screen.setup(width=CANVAS_W, height=CANVAS_H)
screen.tracer(0)
screen.colormode(255)

# Step 4: create the drawing turtle
t = turtle.Turtle()
t.hideturtle()
t.shape("square")
t.shapesize(STAMP_SIZE / 20)
t.penup()
t.speed(0)

# Step 5: draw the decorative background (panel + frame)
print("[Encoder] Drawing background…")
draw_background(t, screen)

# Step 6: draw noise shapes across the canvas
print("[Encoder] Drawing camouflage noise…")
draw_noise_from_list(t, noise_stamps, screen)

# Step 7: stamp each character of the message
print("[Encoder] Stamping message…")
for index, char in enumerate(message):
    stamp_character(t, char, index)
    screen.title("Visual Encoder  //  stamping character " +
                 str(index + 1) + " of " + str(len(message)))
    print("  [" + str(index) + "] '" + char + "'  ASCII=" + str(ord(char)))

# Step 8: stamp the black end-of-message marker
end_x, end_y = grid_position(len(message))
t.penup()
t.goto(end_x, end_y)
t.color("#000000")
t.stamp()

# Step 9: final screen refresh
screen.update()
screen.title("Visual Encoder  //  saving PNG…")

# Step 10: build pixel array and write PNG
print("[Encoder] Building pixel data…")
pixels = build_pixel_array(message, noise_stamps)
print("[Encoder] Writing PNG…")
write_png(OUTPUT_FILE, pixels)

# Step 11: show styled success dialog
screen.title("Visual Encoder  //  done")
show_success_dialog(OUTPUT_FILE)

turtle.done()
if name == "main": main()

####################################################################DECODE####################################################### """
VISUAL DECODER — decode.py PLTW Computer Science | Unit 2 Activity 2.3.1
SETUP (run once in VS Code terminal): pip install pillow

HOW TO RUN: python decode.py

REQUIRES: encoded_message.png must be in the same folder
"""

import tkinter as tk from tkinter import messagebox from PIL import Image

── CONSTANTS (must exactly match encode.py) ────────────────
STAMP_SIZE = 20 # size of each character square in pixels GRID_COLS = 20 # characters per row (same as encoder) START_X = -390 # turtle x-coordinate of the first stamp START_Y = 290 # turtle y-coordinate of the first stamp CANVAS_W = 800 # canvas width used by the encoder CANVAS_H = 600 # canvas height used by the encoder INPUT_FILE = "encoded_message.png"

── FUNCTION: convert character index to pixel coordinates ───
def grid_position_to_pixel(index): """ Converts a character's index (0, 1, 2, ...) to the (x, y) pixel position inside the PNG image.

The encoder used turtle coordinates (origin = center of window).
PIL (Pillow) uses image coordinates (origin = top-left corner).

Conversion formulas:
    px_x = CANVAS_W / 2 + turtle_x
    px_y = CANVAS_H / 2 - turtle_y   (y-axis is flipped)
"""
col      = index % GRID_COLS                     # column (0 to 19)
row      = index // GRID_COLS                    # row (0, 1, 2, ...)
turtle_x = START_X + col * STAMP_SIZE            # turtle x position
turtle_y = START_Y - row * STAMP_SIZE            # turtle y position
px_x     = int(CANVAS_W / 2 + turtle_x)         # PIL pixel column
px_y     = int(CANVAS_H / 2 - turtle_y)         # PIL pixel row
return (px_x, px_y)
── FUNCTION: convert a pixel color back to a character ──────
def pixel_to_char(r, g, b): """ Reverses the encoder's ascii_to_rgb() function.

The encoder stored:  r = ASCII_value * 2
So the decoder uses: ASCII_value = r // 2

Returns None if ASCII value is 0 (the end-of-message marker).

Parameters:
    r, g, b - the red, green, blue values of the pixel (0-255)
"""
ascii_code = r // 2             # reverse of  r = code * 2
if ascii_code == 0:
    return None                 # ASCII 0 = end-of-message marker
return chr(ascii_code)          # chr() is the inverse of ord()
                                # chr(65) = 'A', chr(72) = 'H', etc.
── FUNCTION: read the PNG and extract the hidden message ─────
def decode_image(filepath): """ Opens the encoded PNG image and scans each grid position to reconstruct the original message.

Process:
    1. Open the PNG with Pillow.
    2. Loop through grid positions starting at index 0.
    3. For each position, convert index → pixel coordinates.
    4. Read the pixel's red channel.
    5. Apply r // 2 to get the ASCII value.
    6. Convert ASCII to character with chr().
    7. Stop when ASCII 0 (the end marker) is found.
    8. Join all characters into a string and return it.

Parameters:
    filepath - path to the encoded PNG file

Returns:
    The decoded message as a string.
"""
# Open the image and make sure it is in RGB mode
img    = Image.open(filepath).convert("RGB")
pixels = img.load()              # creates a pixel-access object
width, height = img.size         # get image dimensions for bounds checking

message = []    # list to accumulate decoded characters
index   = 0     # current character slot

while True:
    px_x, px_y = grid_position_to_pixel(index)

    # Stop if the pixel position falls outside the image
    if px_x < 0 or px_y < 0 or px_x >= width or px_y >= height:
        print("[Decoder] Reached edge of image at slot " + str(index))
        break

    # Read the pixel color at this grid position
    r, g, b = pixels[px_x, px_y]

    # Convert color back to a character
    char = pixel_to_char(r, g, b)

    # None means we hit the end-of-message marker
    if char is None:
        print("[Decoder] End marker found at slot " + str(index))
        break

    message.append(char)
    print("  [" + str(index) + "] pixel=(" + str(r) + "," + str(g) + "," + str(b) + ")  char='" + char + "'")
    index += 1

# Join the list of characters into one string
return "".join(message)
── FUNCTION: show the decoded message in a pop-up window ────
def show_decoded_message(message): """ Displays the decoded message in a Tkinter dialog box so the receiver can read it clearly.

Parameters:
    message - the decoded text string to display
"""
root = tk.Tk()
root.withdraw()                  # hide the blank root window
messagebox.showinfo(
    title   = "Decoded Message",
    message = "Message decoded successfully!\n\n" + message
)
root.destroy()
── MAIN PROGRAM ─────────────────────────────────────────────
def main():

print("[Decoder] Opening: " + INPUT_FILE)

# Try to open and decode the image
try:
    message = decode_image(INPUT_FILE)
except FileNotFoundError:
    print("[Decoder] ERROR: '" + INPUT_FILE + "' not found in this folder.")
    print("          Make sure the sender ran encode.py and sent you the PNG file.")
    # Show error dialog
    root = tk.Tk()
    root.withdraw()
    messagebox.showerror(
        "File Not Found",
        "Could not find:  " + INPUT_FILE + "\n\n"
        "Make sure the encoded image is in the same folder as decode.py."
    )
    root.destroy()
    return

print("[Decoder] Decoded message: '" + message + "'")

# Display the result to the user
show_decoded_message(message)
Run main() only when this file is executed directly
if name == "main": main()
