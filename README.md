# SFUD (Serial Flash Universal Driver) Library

---

## 0、SFUD What is it

[SFUD](https://github.com/armink/SFUD) It is an open source serial SPI Flash universal driver library. Since there are most types of serial Flash in the market, and the specifications and commands of each Flash are different, SFUD is designed to solve these differences in Flash, so that our products can support Flash of different brands and specifications, and improve the Flash involved. The reusability and extensibility of the functional software can also avoid the risk of Flash out of stock or discontinuation of the product.

- Main features: support SPI/QSPI interface, object-oriented (support multiple Flash objects at the same time), flexible tailoring, strong scalability, support for 4-byte addresses
- Resource occupation
  - Standard occupancy：RAM:0.2KB ROM:5.5KB
  - Minimum occupancy：RAM:0.1KB ROM:3.6KB
- Design ideas：
  - **What is SFDP** : It is the parameter table standard of serial Flash function formulated by JEDEC (Solid State Technology Association), the latest version V1.6B ([Click here to view](https://www.jedec.org/standards-documents/docs/jesd216b)）The standard stipulates that there will be a parameter table in each Flash, which stores the Flash capacity, write granularity, erase command, address mode and other Flash specification parameters. At present, except for some manufacturers’ older Flash models that do not support this standard, most of the other new factory’s Flash already support the SFDP standard. Therefore, the library will first read the SFDP table parameters when it is initialized.
  - **What to do if SFDP is not supported** ：If the Flash does not support the SFDP standard, SFUD will query the configuration file ( [`/sfud/inc/sfud_flash_def.h`](https://github.com/armink/SFUD/blob/4bee2d0417a7ce853cc7aa3639b03fe825611fd9/sfud/inc/sfud_flash_def.h#L116-L142) ) Whether the Flash is supported in the **Flash parameter information table** provided in. If it is not supported, you can add the parameter information of the Flash in the configuration file (see [2.5 Adding a Flash that is not currently supported by the library] (#25-Adding a flash that is not currently supported by the library)). After obtaining the specifications and parameters of the Flash, all operations on the Flash can be realized.

## 1, Why you choose SFUD

- Avoid project risks caused by Flash out of stock, Flash discontinuation or product expansion;
- More and more projects store firmware in serial Flash, such as: ESP8266 firmware, motherboard BIOS and firmware in other common electronic products, etc. However, various Flash specifications and commands are not uniform. Using SFUD can avoid the problem of not being able to adapt to different types of Flash hardware platforms based on the same functional software platform, and improve the reusability of the software;
- Simplify the software process and reduce the difficulty of development. Now only need to configure SPI communication, you can start playing serial Flash freely;
- Can be used to make Flash programmer/writer

## 2、SFUD how to use

### 2.1 Supported Flash 

The following table shows all Flashes that have been tested on the Demo platform. The flash shown as **does not support** the SFDP standard has been defined in the Flash parameter information table, and more flashes that do not support the SFDP standard need to be improved and maintained together in the future.**  **([Github](https://github.com/armink/SFUD)|[OSChina](http://git.oschina.net/armink/SFUD)|[Coding](https://coding.net/u/armink/p/SFUD/git))**

If you think this open source project is great, you can click [Project Homepage](https://github.com/armink/SFUD) **Star** in the upper right corner, and recommend it to more friends in need.

|Model|Manufacturer|Capacity|Maximum speed|SFDP standard|QSPI mode|Remarks|
|:--:|:----:|:--:|:--:|:--:|:--:|----|
|[W25Q40BV](http://microchip.ua/esp8266/W25Q40BV(EOL).pdf)|Winbond|4Mb|50Mhz|不支持|双线|已停产|
|[W25Q80DV](http://www.winbond.com/resource-files/w25q80dv_revg_07212015.pdf)|Winbond|8Mb|104Mhz|支持|双线||
|[W25Q16BV](https://media.digikey.com/pdf/Data%20Sheets/Winbond%20PDFs/W25Q16BV.pdf)|Winbond|16Mb|104Mhz|不支持|双线| by [slipperstree](https://github.com/slipperstree)|
|[W25Q16CV](http://www.winbond.com/resource-files/da00-w25q16cvf1.pdf)|Winbond|16Mb|104Mhz|支持|未测试||
|[W25Q16DV](http://www.winbond.com/resource-files/w25q16dv%20revk%2005232016%20doc.pdf)|Winbond|16Mb|104Mhz|支持|未测试| by [slipperstree](https://github.com/slipperstree)|
|[W25Q32BV](http://www.winbond.com/resource-files/w25q32bv_revi_100413_wo_automotive.pdf)|Winbond|32Mb|104Mhz|支持|双线||
|[W25Q64CV](http://www.winbond.com/resource-files/w25q64cv_revh_052214[2].pdf)|Winbond|64Mb|80Mhz|支持|四线||
|[W25Q128BV](http://www.winbond.com/resource-files/w25q128bv_revh_100313_wo_automotive.pdf)|Winbond|128Mb|104Mhz|支持|四线||
|[W25Q256FV](http://www.winbond.com/resource-files/w25q256fv%20revi%2002262016%20kms.pdf)|Winbond|256Mb|104Mhz|支持|四线||
|[MX25L3206E](http://www.macronix.com/Lists/DataSheet/Attachments/3199/MX25L3206E,%203V,%2032Mb,%20v1.5.pdf)|Macronix|32Mb|86MHz|支持|双线||
|[KH25L4006E](http://www.macronix.com.hk/Lists/Datasheet/Attachments/117/KH25L4006E.pdf)|Macronix|4Mb|86Mhz|支持|未测试| by [JiapengLi](https://github.com/JiapengLi)|
|[KH25L3206E](http://www.macronix.com.hk/Lists/Datasheet/Attachments/131/KH25L3206E.pdf)|Macronix|32Mb|86Mhz|支持|双线||
|[SST25VF016B](http://ww1.microchip.com/downloads/en/DeviceDoc/20005044C.pdf)|Microchip|16Mb|50MHz|不支持|不支持| SST 已被 Microchip 收购|
|[M25P40](https://www.micron.com/~/media/documents/products/data-sheet/nor-flash/serial-nor/m25p/m25p40.pdf)|Micron|4Mb|75Mhz|不支持|未测试| by [redocCheng](https://github.com/redocCheng)|
|[M25P80](https://www.micron.com/~/media/documents/products/data-sheet/nor-flash/serial-nor/m25p/m25p80.pdf)|Micron|8Mb|75Mhz|不支持|未测试| by [redocCheng](https://github.com/redocCheng)|
|[M25P32](https://www.micron.com/~/media/documents/products/data-sheet/nor-flash/serial-nor/m25p/m25p32.pdf)|Micron|32Mb|75Mhz|不支持|不支持||
|[EN25Q32B](http://www.kean.com.au/oshw/WR703N/teardown/EN25Q32B%2032Mbit%20SPI%20Flash.pdf)|EON|32Mb|104MHz|不支持|未测试||
|[GD25Q16B](http://www.gigadevice.com/product/detail/5/410.html)|GigaDevice|16Mb|120Mhz|不支持|未测试| by [TanekLiang](https://github.com/TanekLiang) |
|[GD25Q32C](http://www.gigadevice.com/product/detail/5/410.html)|GigaDevice|32Mb|120Mhz|不支持|未测试| by [gaupen1186](https://github.com/gaupen1186) |
|[GD25Q64B](http://www.gigadevice.com/product/detail/5/364.html)|GigaDevice|64Mb|120Mhz|不支持|双线||
|[S25FL216K](http://www.cypress.com/file/197346/download)|Cypress|16Mb|65Mhz|不支持|双线||
|[S25FL032P](http://www.cypress.com/file/196861/download)|Cypress|32Mb|104Mhz|不支持|未测试| by [yc_911](https://gitee.com/yc_911) |
|[S25FL164K](http://www.cypress.com/file/196886/download)|Cypress|64Mb|108Mhz|支持|未测试||
|[A25L080](http://www.amictechnology.com/datasheets/A25L080.pdf)|AMIC|8Mb|100Mhz|不支持|双线||
|[A25LQ64](http://www.amictechnology.com/datasheets/A25LQ64.pdf)|AMIC|64Mb|104Mhz|支持|支持||
|[F25L004](http://www.esmt.com.tw/db/manager/upload/f25l004.pdf)|ESMT|4Mb|100Mhz|不支持|不支持||
|[PCT25VF016B](http://pctgroup.com.tw/attachments/files/files/248_25VF016B-P.pdf)|PCT|16Mb|80Mhz|不支持|不支持|SST 授权许可，会被识别为 SST25VF016B|
|[AT45DB161E](http://www.adestotech.com/wp-content/uploads/doc8782.pdf)|ADESTO|16Mb|85MHz|不支持|不支持|ADESTO 收购 Atmel 串行闪存产品线|

> Note: In QSPI mode, two lines indicate that two-line fast reads are supported, and four lines indicate that four-line fast reads are supported.
>
> Generally, the FLASH that supports four-line fast read also supports two-line fast read.

### 2.2 API Description

Let me first explain a structure `sfud_flash` that this library mainly uses. Its definition is located in `/sfud/inc/sfud_def.h`. Each SPI Flash corresponds to this structure, and the structure pointer is collectively referred to as the Flash device object below. After the initialization is successful, the common parameters of SPI Flash will be stored in the `sfud_flash->chip` structure. If SPI Flash also supports SFDP, you can also see more comprehensive parameter information through `sfud_flash->sfdp`. Many of the following functions will use the Flash device object as the first input parameter to realize the operation of the specified SPI Flash.

#### 2.2.1 initialization SFUD Library

Will call `sfud_device_init` to initialize all the devices in the Flash device table. If there is only one Flash, you can also use `sfud_device_init` for single initialization.

> **Note**: The initialized SPI Flash defaults to **write protection removed** status. If you need to enable write protection, please use the sfud_write_status function to modify the SPI Flash status.

```C
sfud_err sfud_init(void)
```

#### 2.2.2 Initialize the specified flash device

```C
sfud_err sfud_device_init(sfud_flash *flash)
```

|Parameter                               |Description|
|:-----                                  |:----|
|flash                                   |Flash device to be initialized|

#### 2.2.3 Enable fast read mode (only available when SFUD turns on QSPI mode）

When SFUD turns on QSPI mode, the Flash driver in SFUD supports the use of QSPI bus for communication. Compared with the traditional SPI mode, using QSPI can speed up the reading of Flash data, but when data needs to be written, because the data writing speed of Flash itself is slower than the SPI transmission speed, the data writing speed in QSPI mode is increased and Not obvious.

Therefore, SFUD's support for QSPI mode is limited to fast read commands. Through this function, you can configure the maximum width of the data line actually supported by the QSPI bus used by Flash, such as: 1-wire (default value, that is, the traditional SPI mode), 2-wire, 4-wire.

After setting, SFUD will combine the currently set QSPI bus data line width and go to [QSPI Flash Extended Information Table](https://github.com/armink/SFUD/blob/069d2b409ec239f84d675b2c3d37894e908829e6/sfud/inc/sfud_flash_def.h#L149-L177) Match the most suitable and fastest fast read command in, then when the user calls sfud_read(), the command will be sent using the QSPI mode transfer function.

```C
sfud_err sfud_qspi_fast_read_enable(sfud_flash *flash, uint8_t data_line_width)
```

| Parameter       | Description                                  |
| :-------------- | :------------------------------------------- |
| flash           | Flash device                                 |
| data_line_width | QSPI The maximum width of the data line supported by the bus, for example：1、2、4 |

#### 2.2.4 Get the Flash device object

The Flash device table is defined in the SFUD configuration file, which is responsible for storing all the Flash device objects to be used, so SFUD supports multiple Flash devices to be driven at the same time. The configuration of the device table is defined in the `SFUD_FLASH_DEVICE_TABLE` macro definition in `/sfud/inc/sfud_cfg.h`, and the detailed configuration method refers to [2.3 Configuration Method Flash] (#23-Configuration Method)). This method returns the Flash device object through the index value of the Flash device located in the device table, and returns `NULL` if it exceeds the range of the device table.

```C
sfud_flash *sfud_get_device(size_t index)
```

|Parameter                               |Description|
|:-----                                  |:----|
|index                                   |The index value of the flash device in the FLash device table|

#### 2.2.5 Read Flash data

```C
sfud_err sfud_read(const sfud_flash *flash, uint32_t addr, size_t size, uint8_t *data)
```

|Parameter                               |Description|
|:-----                                  |:----|
|flash                                   |Flash device object|
|addr                                    |Start address|
|size                                    |The total size of the data read from the start address|
|data                                    |Data read|

#### 2.2.6 Erase Flash data

> Note: The erasing operation will be aligned according to the erasing granularity of the Flash chip (see the Flash data manual for details, generally the block size. After the initialization is completed, you can view it through `sfud_flash->chip.erase_gran`). Please pay attention to the start The address and the erased data size are aligned according to the erase granularity of the Flash chip, otherwise the erase operation will cause other data to be lost.

```C
sfud_err sfud_erase(const sfud_flash *flash, uint32_t addr, size_t size)
```

|Parameter                               |Description|
|:-----                                  |:----|
|flash                                   |Flash device object|
|addr                                    |Start address|
|size                                    |The total size of erased data from the start address|

#### 2.2.7 Erase all flash data

```C
sfud_err sfud_chip_erase(const sfud_flash *flash)
```

|Parameter                               |Description|
|:-----                                  |:----|
|flash                                   |Flash Device object|

#### 2.2.8 Write data to Flash

```C
sfud_err sfud_write(const sfud_flash *flash, uint32_t addr, size_t size, const uint8_t *data)
```

|Parameter                               |Description|
|:-----                                  |:----|
|flash                                   |Flash Device object|
|addr                                    |Start address|
|size                                    |The total size of data written from the start address|
|data                                    |Data to be written|

#### 2.2.9 Erase first and then write data to Flash

> Note: The erasing operation will be aligned according to the erasing granularity of the Flash chip (see the Flash data manual for details, generally the block size. After the initialization is completed, it can be viewed through `sfud_flash->chip.erase_gran`). Please pay attention to the start address The size of the erased data is aligned according to the erase granularity of the Flash chip, otherwise the erase operation will cause other data to be lost.

```C
sfud_err sfud_erase_write(const sfud_flash *flash, uint32_t addr, size_t size, const uint8_t *data)
```
|Parameter                               |Description|
|:-----                                  |:----|
|flash                                   |Flash Device object|
|addr                                    |Start address|
|size                                    |The total size of data written from the start address|
|data                                    |Data to be written|

#### 2.2.10 Read Flash status

```C
sfud_err sfud_read_status(const sfud_flash *flash, uint8_t *status)
```

|Parameter                               |Description|
|:-----                                  |:----|
|flash                                   |Flash Device object|
|status                                  |Device object|

#### 2.2.11 Write (modify) Flash state

```C
sfud_err sfud_write_status(const sfud_flash *flash, bool is_volatile, uint8_t status)
```

|Parameter                               |Description|
|:-----                                  |:----|
|flash                                   |Flash Device object|
|is_volatile                             |Whether it is easy to get lost, true: easy to get lost, and will be lost after power failure|
|status                                  |Current status register value|

### 2.3 Configuration method

All configurations are located in `/sfud/inc/sfud_cfg.h`, please refer to the configuration description below to choose the configuration that suits your project.

#### 2.3.1 Debug mode

Open/close `SFUD_DEBUG_MODE` macro definition

#### 2.3.2 Whether to use SFDP parameter function

Open/close `SFUD_USING_SFDP` macro definition

> Note: After closing, only the Flash information table provided by the library in `/sfud/inc/sfud_flash_def.h` will be queried. Although this will reduce the adaptability of the software, it reduces the amount of code.

#### 2.3.3 Whether to use the Flash parameter information table that comes with the library

Open/close `SFUD_USING_FLASH_INFO_TABLE` macro definition

> Note: After closing, the library only drives the Flash that supports the SFDP specification, and it will also appropriately reduce the amount of code. In addition, the two macro definitions 2.3.2 and 2.3.3 define at least one, or you can select both.

#### 2.3.4 Neither SFDP nor Flash parameter information table is used

In order to further reduce the amount of code, `SFUD_USING_SFDP` and `SFUD_USING_FLASH_INFO_TABLE` can also be **neither defined**.

At this time, just specify the Flash parameters when defining the Flash device, and then call `sfud_device_init` to initialize the device. Refer to the following code:

```C
sfud_flash sfud_norflash0 = {
        .name = "norflash0",
        .spi.name = "SPI1",
        .chip = { "W25Q64FV", SFUD_MF_ID_WINBOND, 0x40, 0x17, 8L * 1024L * 1024L, SFUD_WM_PAGE_256B, 4096, 0x20 } };
......
sfud_device_init(&sfud_norflash0);
......
```

#### 2.3.5 Flash Device table

If there are multiple Flash in the product, you can add a Flash device table. Modify the macro definition of `SFUD_FLASH_DEVICE_TABLE`, the example is as follows:

```C
enum {
    SFUD_W25Q64CV_DEVICE_INDEX = 0,
    SFUD_GD25Q64B_DEVICE_INDEX = 1,
};

#define SFUD_FLASH_DEVICE_TABLE                                                \
{                                                                              \
    [SFUD_W25Q64CV_DEVICE_INDEX] = {.name = "W25Q64CV", .spi.name = "SPI1"},   \
    [SFUD_GD25Q64B_DEVICE_INDEX] = {.name = "GD25Q64B", .spi.name = "SPI3"},   \
}
```

Two Flash devices are defined above (one is sufficient for most products). The names of the two devices are `"W25Q64CV"` and `"GD25Q64B"`, corresponding to the two SPIs of `"SPI1"` and `"SPI3"` respectively Device name (used when porting the SPI interface, located in `/sfud/port/sfud_port.c`), `SFUD_W25Q16CV_DEVICE_INDEX` and `SFUD_GD25Q64B_DEVICE_INDEX` These two enumerations define the indexes of two devices in the device table. Get the device table through the `sfud_get_device_table()` method, and then use this index value to access the specified device.

#### 2.3.6 QSPI Mode

Open/close `SFUD_USING_QSPI` macro definition

After it is turned on, SFUD will also support Flash connected using QSPI bus.

### 2.4 Porting instructions

The transplantation file is located in `/sfud/port/sfud_port.c`, the `sfud_err sfud_spi_port_init(sfud_flash *flash)` method in the file is the transplantation method provided by the library, in which the SPI read and write drivers for each device (required) and retry are completed The configuration of the number of times (required), retry interface (optional) and SPI lock (optional). For more detailed migration content, please refer to the migration files of each platform in the demo.

### 2.5 Add Flash that is not currently supported by the library 

Here you need to modify `/sfud/inc/sfdu_flash_def.h`. For all supported Flash, see the macro definition of `SFUD_FLASH_CHIP_TABLE`. The flash parameters that need to be prepared in advance are: | Name | Manufacturer ID | Type ID | Capacity ID | Capacity | Write mode | Erase granularity (the smallest unit of erasing) | Command corresponding to the erase granularity |. Here is an example of adding GigaDevice's `GD25Q64B` Flash.

This Flash is an early production model of Zhaoyi Innovation, so it does not support the SFDP standard. First, you need to download its data sheet and find the 3 types of IDs returned by the 0x9F command. The last two-byte IDs are needed here, namely `type id` and `capacity id`. `GD25Q64B` corresponding to these two IDs are `0x40` and `0x17` respectively. The other Flash parameters required above can be found in the data sheet. Here we need to focus on the **write mode** parameter. There are 4 write modes provided by the library itself. For details, see the `sfud_write_mode` enumeration type at the top of the file. , The same Flash can support multiple writing modes at the same time, depending on the situation. For `GD25Q64B`, the supported write mode should be `SFUD_WM_PAGE_256B`, that is, write 1-256 bytes per page. The Flash parameters combined with the above `GD25Q64B` should be as follows:
```
    {"GD25Q64B", SFUD_MF_ID_GIGADEVICE, 0x40, 0x17, 8*1024*1024, SFUD_WM_PAGE_256B, 4096, 0x20},
```

Then add it to the end of the `SFUD_FLASH_CHIP_TABLE` macro definition to complete the library's support for `GD25Q64B`.

### 2.6 Demo

Currently, demos under the following platforms are supported

|Path                             |Platform Description|
|:-----                           |:----|
|[/demo/stm32f10x_non_os](https://github.com/armink/SFUD/tree/master/demo/stm32f10x_non_os) |STM32F10X Bare metal platform|
|[/demo/stm32f2xx_rtt](https://github.com/armink/SFUD/tree/master/demo/stm32f2xx_rtt)  |STM32F2XX + [RT-Thread](http://www.rt-thread.org/) Operating system platform|
|[/demo/stm32l475_non_os_qspi](https://github.com/armink/SFUD/tree/master/demo/stm32l475_non_os_qspi) |STM32L475 + QSPI Mode bare metal platform|

### 2.7 License

Adopt MIT open source agreement, please read the contents of the LICENSE file in the project for details.
