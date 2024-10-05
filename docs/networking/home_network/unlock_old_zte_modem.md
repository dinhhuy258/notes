# Unlock old ZTE modem

## Change serial number in unlock ZTE modem

1. **Download and Install Tools**

   - Download the ZTE modem tools from [here](https://github.com/douniwan5788/zte_modem_tools).

2. **Access the Modem via Telnet**

   - Open your terminal.
   - Execute the following command to open the telnet session:
     ```sh
     python3 zte_factroymode.py --user admin --pass 12345678 --ip 192.168.2.1 --port 80 telnet open
     ```
   - This command will generate the username and password required for the telnet connection.

3. **Connect via Telnet**

   - In the terminal, type:
     ```sh
     telnet
     ```
   - Then, connect to the modem by entering:
     ```sh
     open 192.168.2.1
     ```
   - Use the credentials generated in the previous step to log in.

4. **Change the Serial Number**

   - To change the modem's serial number to `ZTEG00e0c8d7`, execute the following commands in the telnet session:
     ```sh
     setmac 1 512 ZTEG00e0c8d7
     setmac 1 2177 00e0c8d7
     reboot
     ```

5. **Verify the Change**

   - After the modem reboots, log back into the modem's interface.
   - Navigate to the modem information section to verify that the serial number has been updated successfully.

