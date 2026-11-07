# Miata Alignment Desk

An installable browser app for recording garage alignment measurements. It saves vehicle details in the browser and exports sessions as JSON.

## Publish and open it on a Galaxy S21

1. Publish this folder on a static web host that serves it over HTTPS. [DEPLOY.md](DEPLOY.md) includes a GitHub Pages workflow and setup steps.
2. Open the hosted page in Chrome on the S21. Tap **Enable phone sensor** before using camber, caster, or sensor toe.
3. Use Chrome's **Add to Home screen** menu to install it. The app caches its shell for offline use after the first visit.

## Measurement methods

- **Camber:** Mount the phone rigidly against the outer rim with the screen facing outward. Keep the same contact points for each reading. The app estimates the wheel angle from the gravity vector and averages a short sensor window. Its two-position calibration uses a verified zero-camber reference; do not calibrate on an unknown wheel angle. Positive camber means the top of the wheel leans outward.
- **Sensor toe:** Follow six guided yaw captures: left rear, right rear, left rear, left front, right front, left front. The app calculates front and rear total toe from shortest circular heading differences, then estimates individual front toe relative to same-side rear readings. Use a rigid adapter with parallel edges and repeatable placement.
- **String or laser-line toe:** Set parallel reference lines against the vehicle centerline. Measure the outboard line gap at matching front and rear points on each wheel, and enter the actual span between those points. A laser level can help align or check the line, but it must be aligned to the car; self-leveling does not find the vehicle centerline. Positive toe is shown as toe-in.
- **Caster:** Keep the phone mounted to a front wheel and capture camber at equal left and right front-wheel steering angles. This is an estimate and depends on the accuracy of the wheel angle.

Phone heading can shift near steel or other magnetic sources, and the browser sensor readings have not been validated against an alignment rack on your S21. A photo-based angle feature is not included because perspective and lens distortion make an uncalibrated picture a weak angle reference. A reliable future photo method would need a known calibration target, a plumb/laser reference in the wheel plane, and camera distortion correction. Use the laser to set or verify reference lines, cross-check sensor toe with line measurements, and verify final settings with calibrated alignment equipment before road or track use.

The Setup screen's **How to measure, and what the math means** section explains the measurement process and formulas. See [REVERSE-ENGINEERING.md](REVERSE-ENGINEERING.md) for the detailed static analysis of the supplied Gyraline Android package.
