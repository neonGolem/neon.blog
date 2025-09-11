+++
title = "I2c Electric Boogaloo"
date = "2025-09-11T23:07:32+03:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = "neon.M@ks"
tags = ["volt.OS", "drunken ramblings"]
keywords = ["placeholder", "i2c", "embedded", "firmware"]
description = "The actual I2C programming lesson"
+++

## Communication established. Now what?

The last chapter details the pains of getting the silicon to respond to us. What needs to happen next is a two way communication with meaning. Since we're working with a microcontroller, the first step is to establish a communication method that enables the MCU to talk to the target in such a way that we don't have to hard-code every single thing. Up til now, I've set it to just fire a command at the I2C bus on the push of a button and then report its' findings over UART. The better way of doing this is to have the machine read the incoming data from UART and react accordingly. 

## The interpreter

What even is an interpreter? In this context, we're talking about a method for the machine to understand what we're saying. A piece of software to translate our words to the MCU. In the case of the RP2040, this is actually very easy. If you're familiar with the CDC-ACM gadget, it essentially takes the emulation of the UART protocol over USB and treats it as stdio. Reading from and writing to it therefore becomes as easy as interacting with the bash shell. If printf writes to stdout, anything incoming is going to be there in stdin. The mechanics of this are actually quite complicated and I'm not going into that now (partly because I dont' entirely understand it myself), but for us right now, this means we can simply parse a buffer char by char, then do some math on it and use the results in a follow-up. 
The code snippet to verify this is as follows:
<pre>
int main() {
    stdio_init_all();
    char buf[MAX_LEN];
    int idx = 0;

    while (true) {
        sleep_ms(100);
        int c = getchar_timeout_us(0);
        if (c != PICO_ERROR_TIMEOUT) {
            if (c == '\n' || c == '\r' || idx >= MAX_LEN - 1) {
                char *send_cmd= strtok(buf, " ");
                char *arg = strtok(NULL, " ");
                printf("read cmd: %s read arg %s\n", send_cmd, arg);
                buf[idx] = '\0';   
                idx = 0;          
            } else {
                buf[idx++] = (char)c;
            }
        }
    }
}
</pre>

The condition for the if statement looks for either \n or \r, which is different between Linux and UNIX. Since my laptop uses \r by default when pressing enter, I need to add that there or the machine would never realise I sent it a string with an endline terminator. 

Let's see if it works then, shall we?

![send UART](/img/send-uart-pls.png)  

Looks like it's doing what we want it to be doing. This is excellent.  
That's it for today though. This post is a placeholder for a few days. Until I clean up my head and write something meaningful. I wanted this up for some of my friends to have something to read while I'm preparing for a proper release.

