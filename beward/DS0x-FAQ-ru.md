# FAQ о домофонах Beward ds05, ds06, ds07 (различных модификаций)

## Содержание



### Аппаратная часть

- [Способы подключения и монтажа](#installation)
- [Процессор (SoC)](#soc)
- [Сенсор](#sensor)
- [Ночная подсветка, IRCUT и датчик освещенности](#night)
- [Кнопка вызова](#call_button)
- [UART](#uart)
- [Дополнительные реле](#other)

### Оригинальная прошивка

- [U-boot](#uboot)
- [Telnet в стоковой прошивке](#telnet)
- [Снятые показатели ipctool](#ipctool)


## Процессор и память (SoC) <a name="soc"></a>

На домофонах серии DS05 и DS06 используется SoC Hisilicon первого поколения — 3518CV100, ядро 3.0.8. Память на отдельном чипе, 128Мб.

На домофонах серии DS07  — 3516CV500, ядро 4.9.37. Память 512Мб.

Флеш память на DS05/DS06 — nor 16Мб, на DS07 — nand 128Мб.

## Сенсор <a name="sensor"></a>

| Модель | Шина | Сенсор                      | Разрешение картинки   |
|--------|------|-----------------------------|-----------------------|
| DS05   | i2c  | Sony imx138                 | 1280x720              |
| DS06   | i2c  | Sony imx225                 | 1280x720 или 1280x960 |
| DS07   | i2c  | Sone imx307 или imx323      | 1920x1080             |

## Ночная подсветка, IRCUT и датчик освещенности <a name="night"></a>

Датчик освещенности:
- DS05/DS06: gpio 3
- DS07: 25

IRCUT:
- DS05/DS06: gpio 6
- DS07: 38

Ночная подсветки на DS05/DS06 реализуется двумя способами, т.к. подключена к ноге, которая может работать как обычное GPIO, так и ШИМ, в этом случае можно управлять яркостью подсветки. При включении как GPIO яркость максимальная.

## Кнопка вызова <a name="call_button"></a>

- DS05/DS06: gpio 4
- DS07: gpio 66


## U-boot] <a name="uboot"></a>

U-boot не заблокирован, в этом плане компания молодец. Блокировать U-boot — это глупость, которая ничего не даёт и только вредит продвинутым пользователям.

DS05/DS06, они примерно одинаковые:

```
hisilicon # printenv
bootdelay=1
baudrate=115200
bootfile="uImage"
ethaddr=00:01:02:03:04:05
filesize=90000
fileaddr=82000000
gatewayip=192.168.55.1
netmask=255.255.255.0
serverip=192.168.55.252
ipaddr=192.168.55.100
bootargs=mem=64M console=ttyAMA0,115200 root=/dev/mtdblock1 rootfstype=jffs2 mtdparts=hi_sfc:3M(boot),12M(rootfs),1M(param)
bootcmd=sf probe 0;sf read 0x82000000 0x80000 0x280000;bootm 0x82000000
stdin=serial
stdout=serial
stderr=serial
verify=n
ver=U-Boot 2010.06 (Jan 09 2013 - 13:05:11)

Environment size: 486/262140 bytes
```

DS07:
```
hisilicon # printenv
arch=arm
baudrate=115200
board=hi3516cv500
board_name=hi3516cv500
bootargs=mem=168M console=ttyAMA0,115200 root=/dev/mtdblock2 rootfstype=yaffs2 rw mtdparts=hinand:512K(boot),3584K(kernel),124M(rootfs)
bootcmd=nand read 0x82000000 0x80000 0x380000;bootm 0x82000000
bootdelay=2
cpu=armv7
ethact=eth0
ipaddr=192.168.0.245
serverip=192.168.0.88
soc=hi3516cv500
stderr=serial
stdin=serial
stdout=serial
vendor=hisilicon
verify=n

Environment size: 460/131068 bytes

```

## Telnet в стоковой прошивке <a name="telnet"></a>

Операция относительно безопасная, но делать аккуратно! Надеюсь, что бевардщики не станут вредительствовать в новых своих устройствах из-за рассказанного простого сброса пароля. В общем, просто надо дописать к вашему bootargs init=/bin/sh, у меня так:

```
setenv bootargs 'mem=64M console=ttyAMA0,115200 root=/dev/mtdblock1 rootfstype=jffs2 mtdparts=hi_sfc:3M(boot),12M(rootfs),1M(param) init=/bin/sh
```

Проверяем через printenv, далее saveenv, reset и прошивка при загрузке вывалится в консоль. Там работает passwd, сбрасываем на нужный нам пароль. Перезапускаем, опять в uboot, возвращаем старый bootargs, загружаемся в прошивку, там уже можно залогиниться спокойно в консоль.

В штатной прошивке есть telnetd, можно запустить и далее работать без UART.

## Снятые показатели ipctool <a name="ipctool"></a>

`ipctool reginfo` на стоке:
- [DS06](DS06-ipctool-reginfo.txt)
- [DS07](DS07-ipctool-reginfo.txt)

Регистры сенсора, со стока эталон:
- [imx138 (DS05)]
- [imx225 (DS06)](imx225_regs.txt)
- [imx307 (DS07)]
- [imx323 (DS07)]

