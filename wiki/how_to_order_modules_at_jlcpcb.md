# How to order Module17s at JLCPCB

The minimum order quantity at JLCPCB is 5 pieces. To order Module17s you will need:
* [a zipped set of gerber files](https://github.com/M17-Project/Module_17/blob/main/hardware/r0.1d/jlcpcb/GERBER-M17-SmartMic.zip),
* [a Bill of Materials](https://github.com/M17-Project/Module_17/blob/main/hardware/r0.1d/jlcpcb/assembly/BOM-M17-SmartMic.csv) (BOM),
* [a pick-and-place file](https://github.com/M17-Project/Module_17/blob/main/hardware/r0.1d/jlcpcb/assembly/CPL-M17-SmartMic.csv)

Gerber files define the board layout and overall design (holes, geometry etc.). A BOM lists all parts used in the device. The last file contains XY coordinates of every part listed in the BOM for automated placing.

Download all the files listed above. For each file, right click on the **Raw** button and select _Save file as_. When you are done, go to [jlcpcb.com](https://cart.jlcpcb.com/quote).

![add gerbs](/assets/img/add_gerbs.png)
Upload zipped gerber files using the _Add gerber file_ button. After the system is done processing your file, you should see that the board info has been updated.

![board_info](/assets/img/board_info.png)
The next step is to select correct copper layers to form the stack. Use the drop-down lists to set it exactly like in the image below. You can leave the rest of the settings with their default values.

![stack](/assets/img/stack.png)
Now it's time to set up the assembly process. Click the slide switch to start.

![assy](/assets/img/assy.png)
Accept the default settings - we want JLCPCB to assemble the top side where all components are located.

![assy2](/assets/img/assy2.png)
After all the settings are confirmed, you will be asked to upload the BOM and pick-and-place files.

![bom_cpl](/assets/img/bom_cpl.png)
Use the files downloaded from GitHub. _CPL_ is how JLCPCB call our pick-and-place file. We recommend selecting the DIY category for the project. Proceed with **Next**.

![bom_cpl_rdy](/assets/img/bom_cpl_rdy.png)
You should be presented with a long list of parts.

![list](/assets/img/list.png)
Ideally, no parts should be listed as _part shortage_. Click the **Next** button below the table. You should now see a 2D render of the board. Don't worry if THT parts are misaligned. In my case the DE9 connector was facing the wrong direction. Don't fret - those parts are assembled manually. After you are done playing with the 3D preview, press the **Save to cart** button.

![preview](/assets/img/preview.png)
Proceed with the payment. This ends the JLCPCB part. Still, a handful of components has to be placed manually. Those parts are:

* OLED,
* volume potentiometer (logarithmic),
* 2 digital potentiometers

Let's start from the top - OLED. This part is manufactured by Waveshare and the module's name is _Waveshare 1.3inch OLED Display Module_. Use your favourite search engine to find those online. Volume pot is a little bit more tricky - the exact value that's used in Module17 might not be available. The schematic asks for a 50k logarithmic. Use any value in the 10..100k range, for example _Bourns PTR901-2025F-A103_. Note that the potentiometer should have a SPST switch built-in. Lastly - SMD digital potentiometers. There are 2 of them in the device - `U8` and `U9`.