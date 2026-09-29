# Soft Landing Lab — equations

A hopper falls in a vertical plane under constant Earth gravity and lands by thrust alone. There is no atmosphere, no ground effect, and no rotational inertia. Those are deleted on purpose: at the speeds in this scene they are corrections, and the force law stays Newton's second law plus a rocket.

## State

The simulated point is the center of mass. \(x\) is crossrange, positive to the right, with the pad centered at zero. \(y\) is altitude of the center of mass, positive up. The contact point is the foot, a distance \(d\) below the center of mass. Legs lengthen \(d\) as they deploy; the bell is the contact point until they do.

\[
m = m_\mathrm{dry} + m_\mathrm{fuel}
\]

\[
d = d_\mathrm{bell} + (d_\mathrm{foot} - d_\mathrm{bell})\,\delta
\]

Altitude above the ground is \(h = y - d\).

## Numbers

| Quantity | Value |
| --- | --- |
| \(g\) | 9.81 m/s² |
| \(I_{sp}\) | 311 s |
| \(T_\max\) | 15,600 N |
| \(m_\mathrm{dry}\) | 920 kg |
| \(m_\mathrm{fuel,0}\) | 58 kg |
| Spool time \(\tau\) | 0.28 s |
| Gimbal limit | ±12° |
| Pad half-width | 16 m |
| Soft sink | 3.2 m/s |
| Soft climb | 1.3 m/s |
| Soft drift | 1.8 m/s |
| Soft tilt | 8° |

Initial contact altitude is 52 m, vertical velocity is −2.4 m/s, crossrange is −7 m, and drift is +0.35 m/s. Full-throttle thrust-to-weight starts at 1.63, so the engine can hover or climb. Hover propellant flow is \(m / I_{sp}\), about 18 s of hover on a full load, and about 11 s at full thrust.

## Thrust and mass

Specific impulse is defined by exhaust speed \(v_e = I_{sp}\, g\). The pilot command is binary (hold or release). The engine follows it with a first-order lag, which is the only non-Newtonian element, and it is an actuator model:

\[
\tau \dot u = u_\mathrm{cmd} - u, \qquad u_\mathrm{cmd} \in \{0, 1\}
\]

\[
T = u\, T_\max, \qquad \dot m_\mathrm{fuel} = -\, T / v_e
\]

If a step asks for more propellant than remains, thrust is scaled for that step so the vehicle cannot burn mass it does not have. Gimbal angle \(\theta\) is zero when upright and positive when the nose and the plume tilt toward \(+x\). It is rate-limited toward the stick and restores toward upright when the stick is released. Thrust is applied through the center of mass:

\[
T_x = T \sin\theta, \qquad T_y = T \cos\theta
\]

\[
a_x = T_x / m, \qquad a_y = T_y / m - g
\]

## Integration

Semi-implicit Euler at \(\Delta t = 1/240\,\mathrm{s}\), with mass and thrust taken at the start of the step:

\[
v \leftarrow v + a\,\Delta t, \qquad r \leftarrow r + v\,\Delta t
\]

## Contact

The run ends at \(h \le 0\), or earlier if the tanks are empty and a soft landing is already impossible.

With the engine out, mechanical energy fixes the impact speed. No remaining choice of gimbal can change it:

\[
v_\mathrm{impact} = \sqrt{v_y^2 + 2 g h}
\]

If \(m_\mathrm{fuel} = 0\), \(h > 1\,\mathrm{m}\), and \(v_\mathrm{impact}\) is above the soft-sink limit, the outcome is **OUT OF FUEL**.

Otherwise the deck judges contact:

- **WIN** — on the pad, sink at most 3.2 m/s, climb at most 1.3 m/s, drift at most 1.8 m/s, tilt at most 8°.
- **CRASH** — any other contact while propellant remains (or the last meter, where the energy test is not applied and the deck judge stands).

Holding a hover spends the same propellant flow as any constant-speed descent, because \(T = mg\) in both cases. Falling, then braking late, shortens the time gravity is being cancelled. That is the whole lesson on the screen: delete excess throttle.
