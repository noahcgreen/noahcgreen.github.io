---
layout: page
title: CME Calculator
date: 2023-03-01 00:00:00
description: Analysis and design of a CME steam battery
img: assets/img/CME.png
importance: 1
category: fun
---

I've been having a lot of fun playing the [Space Exploration](https://spaceexploration.miraheze.org/wiki/Main_Page) Factorio mod lately. It adds a lot of complexity to the game and introduces many new challenges which must be
solved in addition to what the vanilla gameplay requires. One of these is the coronal mass ejection (CME) event,
in which the player must defend their base from energy beams by providing an Umbrella defense facility with
a massive amount of electrical power.

I did not understand the power requirements properly the first time around and failed to defend against the first
CME. Fortunately, my base was only lightly damaged, but I decided to make this writeup and a simple calculator to
make it clear how to defend against CME's to come.

For simplicity, I've just gone over calculations for a 500-degree steam battery. There are other ways to satisfy
the energy requirements, but this is the most straightforward and practical one for a beginner IMO.

---

<div class="calculator">
    <form>
        <h3>CME Calculator</h3>
        <table>
            <tr>
                <td><label for="radius">Planet radius</label></td>
                <td><input id="radius" type="number" value="5692" min="0" step="any"></td>
            </tr>
            <tr>
                <td><label for="solar">Solar modifier</label></td>
                <td><input id="solar" type="number" value="100" min="0"></td>
            </tr>
            <tr>
                <td><label for="duration">CME duration (seconds)</label></td>
                <td><input id="duration" type="number" value="120" min="0"></td>
            </tr>
            <tr>
                <td><label for="boilers">Boilers</label></td>
                <td><input id="boilers" type="number" value="1" min="0"></td>
            </tr>
        </table>
        <div>
            <input type="submit"/>
        </div>
    </form>
    <br/>
    <div>
        <h3>Battery Requirements</h3>
        <table>
            <tr>
                <td>Discharge rate</td>
                <td><span id="peak-power">2.28</span> GW</td>
            </tr>
            <tr>
                <td>Capacity</td>
                <td><span id="total-power">182.4</span> GJ</td>
            </tr>
            <tr>
                <td>Steam tanks</td>
                <td><span id="steam-tanks">76</span></td>
            </tr>
            <tr>
                <td>Turbines</td>
                <td><span id="turbines">394</span></td>
            </tr>
            <tr>
                <td>Charging time</td>
                <td><span id="charge-time">4 hours</span></td>
            </tr>
            <tr>
                <td>Charging power</td>
                <td><span id="charge-power">16</span> MW</td>
            </tr>
        </table>
    </div>
</div>

---

## The Problem

Defending against a CME means providing an Umbrella defense facility with enough power to operate over the
two-minute duration of the event. If the Umbrella's power requirements are fully satisfied for the entire two
minutes, the energy beams will be blocked. However, if at any point the Umbrella does not receive enough power,
it will fail. Even a single moment of failure will cause the player's planet to be hit by deadly CME beams, so it's
important to understand the power requirements well.

The amount of energy which the Umbrella requires is dependent on two factors: the radius and solar modifier of
the defending planet. The [wiki](https://spaceexploration.miraheze.org/wiki/Umbrella) describes how to calculate
the peak power draw, which is the maximum amount of power required by the Umbrella halfway through the CME:

$$\text{peak power draw} = \frac{2 \times \text{radius} \times \text{solar modifier}}{5000} \ \text{GW}$$

For example, the homeworld Nauvis has a radius of 5692 and a solar modifier of 100%, so its peak power draw is

$$\frac{2 \times 5692 \times 1}{5000} = 2.2768 \ \text{GW}$$

To put things into perspective, a beginner base on Nauvis might use a double-digit number of megawatts, so this
requires *hundreds* of times more power than a vanilla player is likely used to building.

Now, it is possible to avoid the CME by simply supplying the Umbrella with 2.28 GW of power at all times. However,
there's a significant downside to this: The Umbrella will only consume that much power at the peak of the event,
exactly halfway through its two-minute duration. During the rest of that time, any excess power goes unused. On top
of that, *all* of that power will go unused when there is no CME to defend against. Producing 2.28 GW constantly
requires a *lot* of resources, and it's pretty wasteful to maintain if you won't actively use that much electricity.

A better solution is to store just enough energy to meet the CME requirements in a battery. Excess energy produced
by the base will be used to fill up the battery prior to the CME. When the CME arrives, the Umbrella will draw power
from the battery instead of the base. This means that we can develop the two systems — the base and the battery -
independently of each other and we don't have to worry about over- or under-developing one of them.

## How Much Energy Does a CME consume?

We have a formula for the maximum amount of power consumed by an Umbrella halfway through a CME. However, the defense
facility doesn't actually draw that much at other points in the event. The power consumption follows a parabolic curve;
it consumes zero power at the beginning of the event, ramps up to peak power consumption at the halfway point, then
lowers consumption back to zero at the end of the CME. We can use this to determine exactly how much power is consumed
throughout, which is the amount we'll need to store.

Since the power draw at time $$t$$ is a parabola, it has the form $$p(t) = at^2 + bt + c$$ for some parameters
$$a, b, c$$. Let's say $$T = 120$$ is the duration of the event in seconds. We know the power draw $$p(t)$$ at three
times:

$$
\begin{gathered}
p(0) = 0 \\
p\left( \frac{T}{2} \right) = P \\
p(T) = 0
\end{gathered}
$$

This gives us a system of three equations in the three unknowns $$a, b, c$$, so we can solve for them
and discover that the power draw at time $$t$$ is

$$p(t) = -\frac{4P}{T^2} t^2 + \frac{4P}{T} t \quad \text{(GW)}$$

To get the total power consumption, we integrate this function over the duration of the event:

$$E = \text{total power consumed} = \int_0^T p(t) \mathop{}\!\mathrm{d}t = \frac{2PT}{3} \quad \text{(GJ)}$$

For Nauvis, we'll need $$\frac{2 \times 2.28 \times 120}{3} = 182.4$$ GJ of stored power to operate the Umbrella.
That lines up nicely with what's described in the [wiki](https://spaceexploration.miraheze.org/wiki/Umbrella)!

## Scalable Energy Storage and Production

Now that we know how much energy our battery needs to store, let's talk about how we'll construct it. Vanilla
Factorio players are most likely to be familiar with using accumulators to store excess energy. However, a
single accumulator can only store up to 5 MJ. That means we'd need 36,480 accumulators to store 182.4 GJ! Putting
aside the amount of raw materials necessary to build that many accumulators, consider the amount of time it would
take to build them. Even if you produced one accumulator per second (a single assembler takes ten), it would take
over ten hours to produce that many. Even if the space required to place them was trivial (it is not), think
about how long it would take to place each one (unless you had tens of thousands of construction robots, but then
you need to factor in the time and cost of building those, etc.).

The recommended method for building a CME battery is to use high-temperature steam. Electric boilers effectively
convert excess electricity to 500-degree steam which can be stored in tanks. The steam is converted back into
electricity by steam turbines. 500-degree steam is a much denser form of energy storage than accumulators: a
single turbine converts 60 steam into 5.8 MW of electricity in one second, meaning a single unit of steam stores
0.096 MJ. One tank holds 25,000 steam, or about 2.4 GJ! We can store enough energy for a Nauvis CME in only
$$\frac{182.4}{2.4} = 76$$ tanks.

It's important to consider how many turbines we'll need. We need enough to to satisfy the peak power draw.
(This is where I messed up initially - I used the average power draw instead of the peak.) A single turbine has an
output rate of 5.8 MW, so we'll need $$\frac{P}{0.0058}$$ turbines ($$\frac{2.28}{0.0058} \approx 394$$ for Nauvis).

The final consideration is how many boilers we should use to charge our battery. Technically, we could just use one,
but it would take quite a while. A single boiler produces 100 steam in 2.15 seconds, consuming 5.18 MW
of power as it does. We need $$\frac{60 \times E}{0.0058}$$ steam, so it'll take $$B$$ boilers

$$\frac{60 \times E}{0.0058} \times \frac{2.15}{\text{boilers} \times 100}$$

many seconds to fill the battery. A single boiler for Nauvis would take over 11 hours. Using more boilers cuts down
the charging time, but each boiler requires electricity (and water) to run.

## Recap

Let's put it all together. Suppose we're defending a planet with radius $$R$$ and solar percent $$S$$ against
a CME lasting $$T$$ seconds. We'll use $$B$$ many boilers to charge our battery.
Our battery will need to discharge up to $$P = \frac{2RS}{5000}$$ GW and store $$E = \frac{2PT}{3}$$ GJ. We'll need:

* $$\frac{60 \times E}{0.0058}$$ units of steam, or $$\frac{60 \times E}{0.0058 \times 25000}$$ full tanks
* $$\frac{P}{0.0058}$$ turbines
* $$\frac{2.15}{100 \times B} \times \frac{60 \times E}{0.0058} \times \frac{1}{3600}$$ hours for the battery to charge.
* $$5.18 B$$ MW of excess energy to power the boilers.

<script type="text/javascript">
const inputs = {
    radius: document.getElementById('radius'),
    solar: document.getElementById('solar'),
    duration: document.getElementById('duration'),
    boilers: document.getElementById('boilers')
}

const outputs = {
    peakPower: document.getElementById('peak-power'),
    totalPower: document.getElementById('total-power'),
    steamTanks: document.getElementById('steam-tanks'),
    turbines: document.getElementById('turbines'),
    chargeTime: document.getElementById('charge-time'),
    chargePower: document.getElementById('charge-power'),
}

const results = {
    peakPower: null,
    totalPower: null,
    turbines: null,
    steam: null,
    tanks: null,
    chargeTime: null,
    boilerPower: null
}

function formatHours(t) {
    const hours = Math.floor(t)
    const minutes = Math.ceil((t - hours) * 60)
    return `${hours} ${hours == 1 ? 'hour' : 'hours'}, ${minutes} ${minutes == 1 ? 'minute' : 'minutes'}`
}

function calculateCMERequirements() {
    results.peakPower = 2 * inputs.radius.value * (inputs.solar.value / 100) / 5000
    results.totalPower = (2 / 3) * results.peakPower * inputs.duration.value
    results.turbines = results.peakPower / 0.0058
    results.steam = results.totalPower * 60 / 0.0058
    results.tanks = results.steam / 25000
    results.chargeTime = 2.15 * results.steam / (100 * inputs.boilers.value * 3600)
    results.boilerPower = 5.18 * inputs.boilers.value
    
    outputs.peakPower.textContent = results.peakPower.toFixed(2)
    outputs.totalPower.textContent = results.totalPower.toFixed(2)
    outputs.steamTanks.textContent = Math.ceil(results.tanks)
    outputs.turbines.textContent = Math.ceil(results.turbines)
    outputs.chargeTime.textContent = formatHours(results.chargeTime)
    outputs.chargePower.textContent = results.boilerPower.toFixed(2)
}

calculateCMERequirements()
document.querySelector('form').addEventListener('submit', event => {
    event.preventDefault()
    calculateCMERequirements()
})
</script>

<style type="text/css">
    .calculator h3 {
        text-align: center;
    }
    .calculator table {
        margin-left: auto;
        margin-right: auto;
        border-collapse: separate;
        border-spacing: 1em 0;
    }
    .calculator input {
        width: 5em;
    }
    .calculator > form > div {
        text-align: center;
    }
</style>
