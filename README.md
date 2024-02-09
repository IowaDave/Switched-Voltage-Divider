# Switched-Voltage-Divider
A microcontroller can switch a voltage divider on and off by means of a suitable PNP transistor placed on the *high-side* of the circuit. 

If the voltage on the high side of the circuit exceeds the tolerance of the microcontroller's I/O pin, then the PNP can be driven safely by a second, NPN transistor.

The circuit described in this article takes input from 16-volt DC power supply repurposed from a discarded laptop computer. 

The voltage divider gives a Point-of-Presence between the resistors where a 12-volt potential can be measured when the 16-volt power supply is enabled. I intend to apply this voltage in a future project for high-voltage parallel programming of AVR microcontrollers.

The circuit enables a 5-volt-tolerant I/O pin of an ATmega328P microcontroller to switch the 12-volt Point-of-Presence on and off.

Advantages and disadvantages for using transistors, rather than MOSFETs, will be discussed at the end.

### The Circuit
![Schematic](images/SwDiv.png)<br>
**Figure 1** Schematic Diagram

The diagram depicts an *unloaded* voltage divider. It satisfies requirements because the application in this case does not need to source any *current* at the 12-volt presence. It merely needs to sense the potential.

Q1 is a 2N3906 bipolar-junction PNP-type transistor. The emitter connects to the 16-volt power supply. The collector sources current into the voltage divider when the base voltage is pulled low.

>***Important!***<br>
>When the base is pulled low (meaning, it has a path to ground), current is going to flow out of that base at a voltage level of near 16 volts.
>
>This can cause two bad things to happen.
>
>The first bad thing is to destroy an Arduino, if the PNP base connects directly to an I/O pin. Those pins cannot tolerate much above 5 volts and 40 milliAmps. Q2 protects the Arduino.
>
>The second bad thing is to blow up the transistor. It goes POW! and pieces fly off with too much current through the base. The resistor, R5, limits the current.

Q2 is a 2N3904 bipolar-junction NPN-type transistor. It serves to drive the base of the PNP. The NPN shorts the 16-volt potential of the PNP base to ground when the controller applies 5 volts to the base of the NPN.

The base of the NPN will draw a bit of current from the controller. An Arduino digital I/O pin in OUTPUT mode will source current when it is HIGH. Here again, the R5 resistor limits the current.

### Untangling the Transistor Jargon
They are versatile things, transistors. For professional communications, engineers practice a rich, private jargon when they talk to each other about transistors.

Unfortunately, they tend to pronounce the same jargon to non-professionals. Efficient and precise terms such as "bias" and "reverse" fail to communicate when the listener or reader does not grasp their meaning.

It took me a while. I translate the jargon as follows.

#### PNP transistor
In a *high-side* configuration, the emitter connects to the positive voltage pole of the circuit; that's the 16-volt power supply in this example.

When current flows, it is going to enter through the emitter and flow out through both the base and the collector. That's what the little arrow is there to indicate.

This means the voltage at the emitter must be greater than that at the base and at the collector. 

While not intuitively obvious -- and this took a while for me to understand, it is also true that the voltage at the collector must be greater than that at the base. A base voltage greater than that of the collector would stand in the way of any current flowing out through the base.

When no current can flow out through the base, then none can flow out through the collector, either.

* The entire voltage divider is then de-energized.
* It will act as a pull-down resistor, taking the 12-volt Point-of-Presence down to 0 volts.

When the base is given a path to ground, then current can flow both through the base and through the collector. 

* The voltage divider is then energized.
* 12 volts can be measured at the Point-of-Presence.

#### NPN Transistor
The 2N3904 is in a *low-side* position with respect to the PNP. That is, it stands between the high-voltage potential of the PNP and ground.

This is the usual placement for an NPN transistor used as a switch.

The collector connects to the high-voltage source. In this example, that is the base of the PNP.

When current flows, it is going to enter through both the collector and the base; it will flow out through the emitter, as the little arrow indicates.

This means the voltage at both the collector and the base must be greater than that at the emitter. It means also that the voltage at the base must be greater than that at the emitter, in order for current to flow through the base.

When no current can flow in through the base, then none can flow out through the emitter, either.

* The base of the PNP then has no path to ground.
* The PNP will de-energize the voltage divider.
* A zero voltage will appear at the Point-of-Presence.

When current can flow in through the base, the it can flow also out through the collector, which goes to ground by way of the current-limiting R5 resistor.

* The base of the PNP then has a path to ground.
* 16-volt current energizes the voltage divider.
* A 12-volt potential appears at the Point-of-Presence.

In this way, toggling a 5-volt output pin of an Arduino or other suitable controller can turn a 12-volt signal on and off, without exposing the controller to a voltage greater than it can withstand.

### Advantages and Disadvantages of Transistors
Someone is likely to object that a MOSFET or two would better serve the purpose, for several reasons that touch upon disadvantages of using transistors.

Firstly, bipolar-junction transistors (BJTs) are essentially glorified diodes, which implies a voltage drop across the transistor even in its most conductive, *saturated* condition. By contrast, carefully selected MOSFETs may take almost nothing away.

Secondly, a BJT must necessarily dissipate some current through the base while it is turned "on". A MOSFET does not consume current through its gate, which corresponds to the base of a BJT. This difference matters more when the circuit needs to operate on battery power.

Current is not a concern for the specific case I have in mind, a high-voltage parallel programmer for AVR microcontrollers, because the power supply to be used offers huge capacity and the operating time will be very short. 

I was able to take the transistor's voltage drop into account in the voltage divider. An iterative approach, measuring actual parts, was taken to discover just which resistors to use, in which order, to obtain the desired result.

#### Advantages of Transistors
The part for switching the high-potential side of a circuit will typically be either a PNP transistor or a P-channel MOSFET (PCM). 

PCMs can be more difficult to find and more expensive for hobbyists like me, compared to PNPs which are cheap and plentiful.

Bob Dorr, a famous Blues musician in Iowa, once told me, "The best ability is *avail*ability." PNP transistors shine in that light for this purpose. Even better, I had them on hand already.

Now you know why I used transistors.

### Circuit To-Dos
A few more resistors will likely come into play as my parallel programmer project advances.

1. A 10-K pullup between the PNP's base and the 16-volt source could help to guard against brief, unwanted current flow when the base is in an indeterminate state, such as when the transistor is first powering-up.

2. A current-limiting resistor on the base of the NPN might make sense. It needs less than 1 milliAmp of current there. Any resistor value ranging from 1K to 5K ought to satisfy requirements.

3. A current-limiting resistor might be needed between the Point-of-Presence and whatever connects to it. Or it might not. Parallel programming connects it to the reset pin of the target AVR controller. Reset is an ordinary I/O pin that will be and remain in a high-impedance state. I believe "high-impedance" means no current would flow through the pin in any event.

