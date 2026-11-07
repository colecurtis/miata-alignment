# Gyraline Android package: method, math, and port notes

## What was inspected

The supplied `com.mechly.gyraline_2.2.0.xapk` is for package `com.mechly.gyraline`, version 2.2.0 (version code 19). The XAPK contains a main APK plus configuration splits. The APK bundles a Capacitor web app. Its main JavaScript asset was obfuscated, but the UI strings, sensor handlers, and alignment calculations could be traced from the bundled code.

The APK was extracted and inspected statically. It was not installed or run. A search for a public Gyraline implementation on GitHub did not surface source code to use; the findings below come from the supplied package and the vendor manual. No APK code, images, or other assets are copied into this project.

## The measurement process behind the APK

Gyraline's toe workflow compares wheel headings, so it depends on a rigid rim adapter with parallel edges, repeatable contact points, and the same adapter edge aimed toward the car's front. The device must be held level during each capture. The six-reading axle check starts at the rear and takes three readings per axle: one side, the other side, then the first side again. This lets the app average two left/right comparisons.

The bundled sensor hook reads `deviceorientation` alpha/beta/gamma and `devicemotion` acceleration. For tilt, it derives gravity by subtracting linear acceleration from acceleration including gravity. The camber capture takes a short sample window. Its recovered calculation is:

```text
gravity = accelerationIncludingGravity - acceleration
rawAngle = atan2(gravity.z, sqrt(gravity.x^2 + gravity.y^2))
calibrationOffset = (position1 - position2) / 2
camber = -mean(rawAngle - calibrationOffset)
```

The app asks for two device-calibration readings with the phone in opposite orientations. The stored offset is subtracted before the result is negated for the app's camber sign convention. The APK warns when the calibration offset is over 1.25 degrees.

## Recovered toe calculations

Yaw headings wrap at 360 degrees. The APK uses the shortest signed angular difference instead of subtracting headings directly:

```text
wrap180(x) = ((x + 180) mod 360 + 360) mod 360 - 180
pairTotalToe = -wrap180(leftYaw - rightYaw)
```

For example, 359.9 degrees and 0.1 degrees differ by 0.2 degrees, not 359.8 degrees. The sign reversal maps the shortest left/right yaw difference to the app's positive-for-toe-in convention.

When one side has an extra reading, it averages the first paired comparison and the last paired comparison:

```text
axleTotalToe = (pairTotalToe(first left, first right)
              + pairTotalToe(last left, last right)) / 2
```

The individual-front workflow uses the front axle total, rear axle total, and a yaw reference between a same-side rear wheel and the selected front wheel. For a left or right front reference, set `sideSign` to +1 or -1 respectively:

```text
relative = wrap180(rearYaw - frontYaw)
sideBase = sideSign * rearTotal / 2 + relative
sideOffset = wrap180(sideBase - sideSign * frontTotal / 2)
leftFrontToe  = frontTotal / 2 + sideOffset
rightFrontToe = frontTotal / 2 - sideOffset
```

The website applies this recovered calculation to each same-side reference in its six readings and circularly averages the left-reference and right-reference estimates. The correction uses half the rear axle total; individual rear toe is not independently measured, so this remains a relative estimate rather than a substitute for a vehicle-centerline measurement.

## How the website uses those findings

- **Camber:** The website reads browser gravity data, averages stable samples, stores two calibration captures, and applies `(position1 - position2) / 2` as the offset. Use a verified zero-camber reference plane for that calibration. The offset stays in browser storage and is included in JSON exports.
- **Sensor toe:** The website guides these captures: left rear, right rear, left rear, left front, right front, left front. It averages a short circular heading window, waits for stable heading and upright phone orientation, then calculates front/rear totals and relative individual front toe using the recovered formulas. Users can save the two front-wheel estimates as wheel readings.
- **String or laser-line toe:** This is an independent geometric check: `wheelToe = atan((frontGap - rearGap) / span)`, then left and right values are summed for each axle. The reference can be a string or a laser-aligned line transferred to rim height.
- **Caster:** The website's estimate uses the two opposite front-wheel camber readings and entered wheel-turn angle: `abs(asin((tan(camLeft) - tan(camRight)) / (2 sin(turnAngle)))`. I did not recover a caster calculation that could confidently be attributed to Gyraline, so this is a separate estimate.

The app records and compares readings; it does not select targets for a particular Miata, identify which adjuster to turn, or control an alignment rack. Model year, suspension, load, ride height, and chosen setup affect the correct specifications and adjustment procedure.

## Accuracy strategies and limits

A four-way self-leveling laser can help establish and check a reference plane, but its self-leveling feature does not align it to the vehicle centerline. Align the plane to a chassis/hub-center datum and verify it at both axles. Account for different front/rear track widths instead of assuming the body panels are centered over the hubs. Use a square or plumb target to transfer a floor laser line to the rim measurement height, then record front/rear gaps as in string mode.

An ordinary phone photo is not a reliable angle measurement: camera position, perspective, and lens distortion can make a vertical wheel look tilted. A stronger future photo method would need a rigid target with known dimensions, a plumb or laser line in the wheel plane, a camera held square to that plane, and lens-distortion correction. Camera calibration documentation from [OpenCV](https://docs.opencv.org/4.13.0/dc/dbb/tutorial_py_calibration.html) explains why known reference points and lens calibration are needed for image measurements.

The browser version has not been validated on a Galaxy S21 against a known heading source or an alignment rack. `deviceorientation.alpha` is a browser-provided yaw-like value whose reference and stability depend on sensor fusion and the environment. Steel near the wheel can disturb compass readings. Adapter geometry and repeatable placement are part of the measurement.

The [Gyraline manual](https://gyraline.com/media/tutorials/manuals/gyraline-manual-march-2026.pdf) describes Android operation with G2 sensor hardware. Finding generic phone-sensor code in the APK does not establish that the retail Android workflow normally measures from an S21's built-in sensors. The manual also describes a three-position caster flow (straight ahead and opposite turns); the website's two-turn estimate is a separate approximation.

Browser sensor events are covered by the [W3C Device Orientation and Motion specification](https://www.w3.org/TR/orientation-event/) and [MDN DeviceMotionEvent reference](https://developer.mozilla.org/en-US/docs/Web/API/DeviceMotionEvent). The app must be hosted over HTTPS for browser sensor and PWA service-worker access.
