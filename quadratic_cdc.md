# Quadratic CDC
## The Linear Dosing System
### Description
AguaClara's current dosing system is a linear dosing system. This means that all chemical flow rates are required to linearly increase with respect to head changes. To achieve linearity, we use small straight tubes to increase velocity and ensure major losses dominate the headloss. Additionally, we use a special pipe weir (LFOM) to ensure the plant flow rate also obeys a linear relationship between water height and flow rate. Finally, to correlate the two flow rates (plant and chemical), we use an adjustable balance that can react to changes in plant flow rate to increase or decrease chemical flow rate proportionately, and hence maintain a proper dose. To change the dose, a sliding adjuster is used on the balance to adjust the relative travel of chemical head to plant entrance tank head. In summary, the dosing system relies on the following four components.

### Elements

* Constant head tank (CHT):
  - to maintain a constant head for the chemicals
  - a tank with a float valve
* Major Loss Manifold:
  - to ensure the dosing system is linear by increasing major losses
  - a series of small tubes running in parallel
* Balance:
  - to correlate the entrance tank water height with the available head through the CDC.
  - to enable dose changes with a sliding adjustment.
  - a simple see-saw balance with a float on one end to detect entrance tank water height, and a free-fall at the other end (adjustable) to determine chemical head available.
* Linear Flow Orifice Meter:
  - to ensure entrance tank water height directly corresponds with the plant flow rate.
  - a tube with some holes drilled out in a pattern that approximates a sutro weir.

### Challenges

There have been challenges in both the operation and fabrication of this system:

* Fabrication:
  - Each plant requires **different length tubing**. This has proved to be a point that has caused uncertainty throughout the construction process.
  - Fabricators **don't intuitively understand** the basis of operation, sometimes leading to surprising conclusions.
  - Fabricators take **multiple days** to install this system, and require an engineer to oversee the work. In Zamorano, a 40 L/s plant, the dosing system took a week to construct.
  - Several CDC components are ordered from the USA because they are **not available in Honduras**. For example, a high quality balance can cost more than $150 to fabricate and send from the USA.
* Operation:
  - Operators have to **unclog the whole system regularly**, which can be a tedious process.
  - Operators also **don't intuitively understand** the basis of operation, which often leads to cutting additional holes in LFOMs, increasing the diameter of the major headloss tubes, etc...

From the above challenges, it would seem apparent that a system that is more intuitively understandable would reduce fabrication time and increase operational reliability. The main source of confusion seems to be the LFOM and the major headloss tubes. Operators commonly wonder "why are the holes spaced so strangely in the LFOM?", and "why aren't the tubes larger?" and "can I but a gate valve in to control the chemical flow rate instead?"

## The Quadratic Dosing System

### Description
In an attempt to reduce the complexity and cost of the current AguaClara dosing system, a quadratic dosing system is proposed. This system eliminates the need for a major headloss element and an LFOM by using a quadratic relationship between head and flow rate, which is the governing relationship for a flow through an orifice: ([k_minor is assumed to be 1 in a full expansion](http://www.engr.colostate.edu/~pierre/ce_old/classes/CIVE%20401/projects%202015/Minor%20Losses%20(1).pdf))

\[
H_L = \frac{V^2}{2g} = \frac{Q_{orifice}^2}{A_{orifice}^22g} \implies Q_{orifice} = A_{orifice}\sqrt{H_L2g}
\]

The key idea is to use a small orifice (or collection of orifices) to regulate the chemical flow and the plant flow. If both the flow rate of the plant and the chemicals exhibit a quadratic headloss increase with equivalent flow rate increases, the two will still maintain a constant proportion and hence a constant flow. In this system, instead of varying the balance adjustment to get a different dose, one would vary the orifice area.

### Elements

* Constant head tank (CHT):
  - to maintain a constant head for the chemicals
  - a tank with a float valve
* Minor Loss Valve:
  - ensures minor losses dominate by using small orifices.
  - a concentric sliding tube system.
* Balance:
  - to correlate the entrance tank water height with the available head through the CDC.
  - a simple see-saw balance with a float on one end to detect entrance tank water height, and a free-fall at the other end (adjustable) to determine chemical head available.
* Quadratic Flow Orifice Meter:
  - to ensure entrance tank water height quadratically corresponds with the plant flow rate.
  - a single horizontal orifice.

### Determining Orifice Spacing and Size

To determine the orifice spacing and size, we take in a few user-defined constants.

```python
from aide_design.play import *



def hole_size(q, hl, dose_max, stock_concentration, sliding_height=20*u.cm, n_settings=20):
  """hole_pattern determines the size of holes based on flow rate. Currently supports one row of holes.

  Parameters
  ----------
  q : volume/time
      desired flow rate through all holes.
  hl : length
      the head at which q is desired.
  dose_max : mass/volume
      the max dose desired in the plant.
  stock_concentration : mass/volume
      the stock concentration
  sliding_height : length
      the height over which the control slider will slide.
  n_settings : type
      the total number of settings, which currently corresponds to the current number of holes.

  Returns
  -------
  diameter of holes: length
      the diameter of holes.
  """
  # the total max chemical flow rate required through all the holes
  q_holes = (q*dose_max)/stock_concentration

  # flow rate through one hole
  q_hole = q_holes/n_settings

  # area of a single hole
  a_hole = pc.area_orifice(hl, exp.RATIO_VC_ORIFICE, q_hole)

  # diameter
  d_hole = np.sqrt(a_hole/np.pi) * 2

  # return the result in millimeters.
  return d_hole.to(u.mm)

```

And the beginnings of a test suite:

```python
# For the plantita
hole_size(1*u.L/u.s, 10*u.cm, 60 *u.mg/u.L, 150 * u.g/u.L)
# >>> 0.16988776957493767 millimeter
```

### Drilling The Orifices

To drill the orifice, I used an angled engraving bit. Specifically, the one available [here](https://www.amazon.com/gp/product/B00EQ1X1GY/ref=oh_aui_detailpage_o03_s00?ie=UTF8&psc=1), however, one with a larger shank and narrower taper may be preferable, such as [this](https://www.amazon.com/YIYATOO-3-175mm-Carbide-Engraving-Router/dp/B017LGA36G/ref=pd_sim_469_4?_encoding=UTF8&pd_rd_i=B017LGA36G&pd_rd_r=X67AE3BVN227VZHK9XAP&pd_rd_w=uMR7Q&pd_rd_wg=PE7mp&psc=1&refRID=X67AE3BVN227VZHK9XAP). With just the $1 engraving bit, I could achieve hole sizes varying from 0.1 mm to 3.175 mm (the shaft size). I did this by using a shaft collar with a bit of tubing. The shaft collar would grab the bit in the part without a cutting edge, and a bit of tubing would cover the cutting edge until the desired size was exposed. To determine the vertical component of the cutting edge I would leave exposed, I performed the following calculation:

```python

def L_cutting_edge_exposed(d_hole, angle_engraver, d_engraver, t_pipe):
  """Calculate the length of the cutter exposed.

  Parameters
  ----------
  d_hole : length
      the diameter of the desired hole
  angle_engraver : angle
      usually 10, 20 or 30 degrees. The angle of the engraver for the bit.
  d_engraver : length
      usually 0.1-1 mm. The diameter of the engraver pecified for the bit.
  t_pipe : length
      the thickness of the pipe

  Returns
  -------
  length
      The section that must be exposed.
  """
  if d_hole < d_engraver:
    raise UserWarning("The diameter of the desired hole cannot be smaller than the radius of the engraver.")
  return t_pipe + (d_hole-radius_engraver)/np.tan(angle_engraver)+

# And for the 1 L/s plant (the plantita):
L_cutting_edge_exposed(0.17*u.mm, 20*u.deg, 0.1*u.mm, 1.8*u.mm)
# >>> 1.9923234193618236 millimeter
```

So it is settled, I will drill 20 holes each 1 cm apart each 0.16 mm in diameter.
