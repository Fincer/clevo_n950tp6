This configuration uses a Thermal daemon (`thermald`) to control CPU temperatures. I have found out that without thermal control Clevo N950TP6 CPU temperatures may rise quite high, peaking to 94 celcius. Additionally, CPU fan spins up quite rapidly, creating a lot of noise.

This is not what I want, neither do you. Therefore, I configured thermald exclusively for Clevo N950TP6 laptop so that the maximum allowed temperature for CPU is 72 celcius. The limit is not precise as the thermal daemon can't control temperatures in very detailed level. That's why I would say there is a 5-7 celcius threshold upwards, so the maximum temperature with thermald setting 72C is approximately 78-80C. I have still noticed that the temperature can peak up to over 90C for short times in some cases.

The maximum temperature is set in systemd `thermald-temp.service` file provided here: by default, the temperature value `uint32:75000`. Adapt it for your needs.
