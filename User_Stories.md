# Assignment 4 - User Stories and Use Cases

### Cameron Pocisk and Jake Skaff

Product context: a stylus that can sense the color of a real-world surface and pass that color to drawing software. Every numeric threshold below is an assumption (listed in the Assumptions table) that we will confirm and revise in Week 9. Elicitation was competitive analysis plus simulated stakeholder interviews and one observation, recorded in docs/elicitation/elicitation_notes.md.

## Assumptions

| ID | Assumption | Used in |
|---|---|---|
| A-1 | The stylus keeps at least 20% battery for capture and reset to work. | UC-01, UC-02 |
| A-2 | The sensor works in 200 to 10,000 lux. | UC-01, AC-01.1, AC-01.3 |
| A-3 | Capture and success signal take 2 seconds or less. | AC-01.1 |
| A-4 | Accuracy target: CIEDE2000 color difference of 3 or less against a spectrophotometer reading. | AC-01.1 |
| A-5 | An error is signaled within 1 second when the tip is 5 mm or more from a surface. | AC-01.2 |
| A-6 | A pending capture reaches the drawing software within 5 seconds of reconnection. | AC-01.3 |
| A-7 | A reset finishes within 10 seconds. | AC-02.1, AC-02.2 |
| A-8 | The stylus can store at least 3 palettes and 1 device connection (tests use exactly these counts). | AC-02.1 to AC-02.3 |

## Stakeholder Map

| Category | Stakeholder | What they care about | Story |
|---|---|---|---|
| Primary | Digital illustrator (at a tablet) | Getting a real-world color into their drawing without guessing | US-01 |
| Primary | Illustrator working away from a tablet | Collecting several colors on location and using them later | US-02 |
| Secondary | Art director approving brand illustrations | Confirming captured colors match the brand palette | US-03 |
| Hidden | Artist with color blindness | Knowing the captured color is the one they intended | US-04 |
| Hidden | Art school equipment manager | Lending styluses without one borrower's data leaking to the next | US-05 |

## User Stories

US-01 (primary): As a digital illustrator, I want to capture the color of a real-world surface and use it in my drawing software, so I can stop eyeballing and save time.

US-02 (primary): As an illustrator sketching without my tablet, I want to capture several colors at a time and retrieve them later, so I can create a reference palette and paint from my desk.

US-03 (secondary): As an art director approving brand illustrations, I want each captured color reported as a numeric value, so that I can verify it matches the brand's palette.

US-04 (hidden): As an artist with color blindness, I want each color to be captured with a numeric value and name, so I can confirm I captured the color I intended.

US-05 (hidden): As an art school equipment manager, I want each stylus returned to a default state between borrowers, so that one student's saved palettes and settings do not carry over.

### INVEST Self-Check

Y = passes, Watch = passes with a note to resolve. No story names a UI element (button, menu, screen, or gesture).

| Story | Independent | Negotiable | Valuable | Estimable | Small | Testable | Notes |
|---|---|---|---|---|---|---|---|
| US-01 | Y | Y | Y | Watch | Y | Y | Estimable once we decide which drawing software we support. |
| US-02 | Watch | Y | Y | Y | Watch | Y | Builds on the capture ability from US-01, so schedule after it. Bundles "capture several" and "retrieve later"; split if one half proves large. |
| US-03 | Watch | Y | Y | Y | Y | Y | Shares the numeric output produced for US-01, so schedule after it; it can still be tested alone with a stored reading. |
| US-04 | Watch | Y | Y | Y | Y | Y | Shares the numeric output produced for US-01, so schedule after it. "Name" needs a defined color-name list; negotiable on which list. |
| US-05 | Y | Y | Y | Y | Y | Y | Does not depend on any capture feature; only on saved data existing. |

## Use Cases

### UC-01: Capture Surface Color and Apply It in Drawing Software

Expands: US-01

Primary actor: Digital illustrator

Secondary actors: Drawing software (receives the color)

Preconditions:
1. Stylus battery is at 20% or higher.
2. Stylus is connected to a device running the drawing software.
3. The surface to be sampled is lit between 200 and 10,000 lux (checkable with a lux meter).
4. The drawing software has a canvas open.

Main success flow:
1. Illustrator places the stylus tip against a real-world surface.
2. System detects surface contact.
3. Illustrator triggers a capture.
4. System measures the surface color, converts it to sRGB, and signals success.
5. Illustrator starts a stroke on the canvas.
6. System renders the stroke in the captured color.

Alternate flow A1 (at step 3, stylus not connected to drawing software):
1. Illustrator triggers a capture while the stylus is disconnected.
2. System measures the color and stores it in stylus memory as a pending capture, and signals success.
3. When the connection is restored, system sends the pending color to the drawing software, which makes it the active color. The flow resumes at step 5.

Exception flow E1 (at step 4, no valid reading):
1. System cannot obtain a valid reading because the tip is not in contact with the surface, light is outside the supported range, or the reading is unstable.
2. System discards the reading, signals an error that names the cause, and leaves the drawing software's active color unchanged.
3. Illustrator corrects the cause and returns to step 1, or stops.

Postcondition (success): The drawing software's active color equals the captured value, the capture is stored in the capture history with a timestamp, and the stylus is still connected.

Postcondition (exception E1): The active color and capture history are the same as before the attempt.

### UC-02: Reset Stylus to Default State

Expands: US-05

Primary actor: Art school equipment manager

Secondary actors: None (the previous borrower's leftover data is what gets removed, but the borrower takes no part in the flow)

Preconditions:
1. Stylus has at least one saved palette and at least one setting changed from its default.
2. Stylus battery is at 20% or higher.
3. Manager is holding the stylus.

Main success flow:
1. Manager starts a reset on the returned stylus.
2. System asks for confirmation and states that all saved palettes, settings, and device connections will be erased.
3. Manager confirms.
4. System erases all palettes, restores all settings to defaults, removes all stored device connections, and signals completion.
5. Manager inspects the stylus state.
6. System reports 0 saved palettes and default settings.

Alternate flow A1 (at step 3, manager cancels):
1. Manager cancels instead of confirming.
2. System makes no changes and returns to its previous state.

Exception flow E1 (at step 4, power is lost during reset):
1. Battery drops out or the stylus powers off while erasing.
2. On the next power-up, system detects the reset was incomplete and finishes it before allowing any capture or transfer.
3. System signals when the reset is complete.

Postcondition (success): Stylus holds 0 saved palettes, all settings equal factory defaults, and no device connections are stored.

Postcondition (alternate A1): Stylus data is unchanged.

Postcondition (exception E1, after next power-up): Same as success.

## Acceptance Criteria

### For UC-01

AC-01.1 (main flow)
Given the stylus is at 50% battery and connected to drawing software, and a reference swatch of known color is lit at 500 lux,
When the illustrator presses the tip against the swatch and triggers a capture,
Then within 2 seconds the stylus signals success, and the drawing software's active color is within a color difference (CIEDE2000) of 3 or less from the swatch's spectrophotometer-measured value.

AC-01.2 (exception flow E1)
Given the stylus is connected and the tip is held 5 mm or more away from any surface,
When the illustrator triggers a capture,
Then within 1 second the stylus signals an error that names the cause, the drawing software's active color is unchanged, and the capture history count is unchanged.

AC-01.3 (alternate flow A1)
Given the stylus is disconnected from the drawing software and the tip is on the reference swatch at 500 lux,
When the illustrator triggers a capture and the connection is restored later,
Then the capture history has one new entry, and within 5 seconds of reconnection the drawing software's active color matches that entry.

### For UC-02

AC-02.1 (main flow)
Given a stylus with 3 saved palettes, one changed setting, and 1 stored device connection, at 50% battery,
When the manager starts a reset and confirms,
Then within 10 seconds the stylus reports 0 saved palettes, every setting equals its factory default, and 0 device connections are stored.

AC-02.2 (exception flow E1)
Given a reset is in progress on a stylus with 3 saved palettes,
When power is cut 3 seconds after the reset starts and then restored,
Then within 10 seconds of power-up the reset completes, 0 palettes remain, and no capture or transfer succeeds until it has completed.

AC-02.3 (alternate flow A1)
Given a stylus with 3 saved palettes and the reset confirmation showing,
When the manager cancels,
Then the stylus still holds 3 saved palettes and all settings are unchanged.
