# node-anemometer

[![NPM](https://nodei.co/npm/node-anemometer.png)](https://npmjs.org/package/node-anemometer)

## Install

```sh
npm install node-anemometer
```

```sh
yarn add node-anemometer
```

## Required hardware

**At the moment this library can only be used with an extra counter module [PCF8583](https://www.nxp.com/docs/en/data-sheet/PCF8583.pdf).** It is not quite easy to use it.

### The circuit board

**You need an debounce filter!** Otherwise, the chip will count many noise values. _For more information click [here](https://ptvo.info/zigbee-configurable-firmware-features/external-sensors/pcf8583-zigbee-counter/)._ There are many designs of such PCBs. Here is one example: [circuit board from Horter](https://www.horter.de/doku/i2c-counter-PCF8583_db.pdf)

## System settings

[Configuring I2C on the Raspberry Pi](https://github.com/fivdi/i2c-bus/blob/HEAD/doc/raspberry-pi-i2c.md)

[Creating Multiple I2C Ports](https://www-laub--home-de.translate.goog/wiki/Raspberry_Pi_multiple_I2C_bus?_x_tr_sl=de&_x_tr_tl=en&_x_tr_hl=de&_x_tr_pto=wapp) (_for advanced people_)

## Usage

_Examples in the calculation not adjusted to your anemometer_

### Example for TypeScript (with ES Modules):

```typescript
import { Anemometer, calcFactor, WindSpeed, WindSpeedUnits } from 'node-anemometer';

const calc = (pulses: number, time: number): WindSpeed => {
	// You cannot divide by 0
	if (time <= 0) {
		return new WindSpeed(0, WindSpeedUnits.kilometersPerHour);
	}

	// More about the calculation can you find below
	const windSpeed = (pulses / 2 / time) * calcFactor(9, 1.18);

	// You must always return a class of the type WindSpeed
	return new WindSpeed(windSpeed, WindSpeedUnits.kilometersPerHour);
};

// With the initialization of the class it starts to measure the wind speed and stores it in a cache
// Create a maximum of only one instance per anemometer!
const myAnemometer = new Anemometer(calc);

// Wait 15 seconds to have a usable average value
setTimeout(() => {
	// '.getData()' calculates the average wind speed of the past x seconds
	const data = myAnemometer.getData(10);

	console.log(`Wind speed: ${data.rounded(2)} ${data.unit}`);

	// Herewith you can stop the permanent reading process
	// After that, the class can no longer be used
	myAnemometer.cleanUp();
}, 15000);
```

### Example for JavaScript (with CommonJS):

```js
const { Anemometer, WindSpeed, WindSpeedUnits, calcFactor } = require('node-anemometer');

const calc = (pulses, time) => {
	// You cannot divide by 0
	if (time <= 0) {
		return new WindSpeed(0, WindSpeedUnits.kilometersPerHour);
	}

	// More about the calculation can you find below
	const windSpeed = (pulses / 2 / time) * calcFactor(9, 1.18);

	// You must always return a class of the type WindSpeed
	return new WindSpeed(windSpeed, WindSpeedUnits.kilometersPerHour);
};

// With the initialization of the class it starts to measure the wind speed and stores it in a cache
// Create a maximum of only one instance per anemometer!
const myAnemometer = new Anemometer(calc);

// Wait 15 seconds to have a usable average value
setTimeout(() => {
	// '.getData()' calculates the average wind speed of the past x seconds
	const data = myAnemometer.getData(10);

	console.log(`Wind speed: ${data.rounded(2)} ${data.unit}`);

	// Herewith you can stop the permanent reading process
	// After that, the class can no longer be used
	myAnemometer.cleanUp();
}, 15000);
```

### Calculation

The wind speed is always calculated with the signals and the time. Often you can find information about this in the data sheet. If you can't find any, I will now describe how the calculation works.

The following basic equation applies:

$$windSpeed = \frac{\left ( \frac{pulses}{pulses~per~rotation} \right )}{time}\ast anemometerFactor$$

**`pulses`**: Signals emitted by the anemometer. The are similar when you press a button

**`pulses per rotation`**: Is the number of signals that are output for a complete rotation

**`time`**: Is the duration (in seconds) in which the measurement was performed

**`anemometerFactor`**: Mostly to find in the datasheet, but can also be calculated by yourself. More information below

![](./images/pulses.svg)

### Calculate anemometor factor

The `.calcFactor()` function computes the multiplier for the calculation of the wind speed in km/h. The radius (measured distance from the center to the edge of one of the cups) in cm and the anemometer factor. The anemometer factor is a value to compensate for the lost wind energy when turning the arms. In my case this value is `1.18`.

![](./images/anemometer.svg)

## TypeDoc

Click [here](https://killerjulian.github.io/node-anemometer/)

---

## Useful information and references

**???? Special thanks to [@DasMelone](https://github.com/DasMelone) who helped me to work out this project**

- ???? [Measuring wind speed using Raspberry Pi](https://projects.raspberrypi.org/en/projects/build-your-own-weather-station/5)
- ???? [Why a debounce filter?](https://ptvo.info/zigbee-configurable-firmware-features/external-sensors/pcf8583-zigbee-counter/)
- ???? ???????? [Setting multiple i2c buses on the Raspberry Pi](https://www.laub-home.de/wiki/Raspberry_Pi_multiple_I2C_bus)
- ???????? [Video of the calculation with an Arduino](https://www.youtube.com/watch?v=Mr05UumeQsk)
- ???????? [I2C pulse counter module by Horter](https://www.nikolaus-lueneburg.de/2019/05/i2c-impuls-counter-modul/)
- [Counting events with Arduino and PCF8583](https://tinkerman.cat/post/counting-events-with-arduino-and-pcf8583/)
- [Low Power Counters](https://hackaday.io/project/174898-esp-now-weather-station/log/184063-low-power-counters)
- [What is a Schmitt trigger?](https://en.wikipedia.org/wiki/Schmitt_trigger)
- [Datasheet anemometer](https://www.argentdata.com/files/80422_datasheet.pdf)
- [Datasheet PCF8583](https://www.nxp.com/docs/en/data-sheet/PCF8583.pdf)
- [Datasheet 74HC132](https://assets.nexperia.com/documents/data-sheet/74HC_HCT132.pdf)

_???? - I recommend to read this_

_???????? - Content that is only available in German_

---

## Author

???? **KillerJulian <info@killerjulian.de>**

- Github: [@KillerJulian](https://github.com/KillerJulian)

## ???? Contributing

Contributions, issues and feature requests are welcome!<br />Feel free to check the [issues page](https://github.com/KillerJulian/node-anemometer/issues). You can also take a look at the [contributing guide](https://github.com/KillerJulian/node-anemometer/blob/master/CONTRIBUTING.md).
