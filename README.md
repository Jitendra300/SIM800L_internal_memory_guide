# Introduction
Recently I have been playing a lot with the GSM-SIM800L module which is used for SMS / Calls related stuff. Slowly I learned how much capable this module is and how to use various AT commands. But then I hit a block when I wanted to play some automated voice over the call. I had used DFPlayer mini but I had various issues ranging from background noise (which I had solved by adding ceramic capacitor and some resistance) and sometimes the DFPlayer mini not working at all. Therefore I tried to use SIM800L by placing my audio files into its memory. I had found this freaking awesome [guide](https://github.com/martinhol221/SIM800L_DTMF_control/wiki/Loading-ARM-audio-files-in-the-SIM800L-modem) to help me out the most of my situation. This page tutorial is heavily inspired from it and aims to further simplify the process.

# Components Required
- **USB-TTL module**  
  I had used this as it had largely simplifed the serial communication and transmission of files.
- **SIM800L**  
  I have only tested this in SIM800L and don't know of any other GSM-SIM modules which have such functionality. Also I had heard that only certain SIM800L could have this storage capability. Some say it depends on our firmware version which I am not entirely sure.
- **Jumper wires**  
- **DC-DC Buck Convertor**  
  Cause SIM800L has range of about 3.4V to 4.4V thus we use DC-DC Buck convertor to reduce the 5V output from GSM-SIM800L into somewhere of our SIM800L range.
- **Resistors (1K Ω and 2K Ω)**  
  Cause we have to use one voltage divider at the RX of our GSM-SIM800L.

# Other things required
- **Terminal**  
  Sorry, I am only comfortable with that. For GUI you can refer the [martin guide](https://github.com/martinhol221/SIM800L_DTMF_control/wiki/Loading-ARM-audio-files-in-the-SIM800L-modem).
- **Minicom**  
  For serial communication. You can use other tools like **screen** too but I was very comfortable with minicom.
- **ffmpeg**  
  To convert the .mp3 / .wav files into .amr files which can be uploaded into SIM800L.

# Circuit diagram
Will upload later don't worry! 

# Start
First of all have your voice files in any format which ffmpeg is comfortable (which it obviously will) and then try to convert them into .amr files by using ffmpeg tool.<br><br>
Command to turn any .mp3 files into .amr files is: `ffmpeg -i input.mp3 output.amr`
<br><br>
But to make .amr file size even less use this: `ffmpeg -i input.mp3 -ar 8000 -ab 4.75k -ac 1 output.amr`
<br><br>

Now ones we have the .amr files we can plug in the usb-ttl. Then find the serial port which would be like **/dev/ttyUSB0** or **/dev/ttyACM0** or something similar. To find that just type `/dev/` and then press tab and find it. <br><br>
After finding out lets start our minicom: `sudo minicom -D /dev/ttyXXXX -b 9600` <br><br>
Which means minicom open the serial port at **/dev/ttyXXXX** at **9600** baud rate. <br><br>
Sometimes minicom might not work at this baud rate so try out 115200 nor don't give baud rate at all and try: `sudo minicom -D /dev/ttyXXXX` <br><br>

Now after opening the minicom interface type `AT` and press enter which should return `OK`. Congrats! Its a huge success.

Now to the real part. Lets view if we have files already inside this SIM800L by typing:  `AT+FSLS=C:\` <br><br>
which should just return `User\`. Which means everything here is empty. <br><br>

Now open another terminal cause we would need it in few minutes. Go to the directory where you saved the .amr files and then check its size (in terms of bits) by typing `wc -c file.amr` where file.amr is your amr file, now this should give the no of bits the file is made of. <br><br>

Now go to the minicom terminal window and then write: `AT+FSCREATE=1.amr` <br><br>
then write `AT+FSWRITE=1.amr,0,XXXX,10` <br><br>
where XXXX is the number of bits the file is.
This will give you `>` and now we have to fastly return the file by going into the other terminal and doing `cat 1.amr > /dev/XXXX` and this has created the file. And now to play this file during call all it takes is this command: `AT+CREC=4,"C:\1.amr",0,100` <br><br>

NOTE: You could have also saved the file in **C:\Users** directory.

# Final Thoughts
I had extreme fun changing the design of the whole circuit from using DFPlayer mini to just using GSM-SIM800L. The complexity reduced a lot as we can see in the below images. But there are several drawbacks too like:
- As the files are in .amr format thus the quality reduces,
- Max capacity is 40KB thus can't store big files.
