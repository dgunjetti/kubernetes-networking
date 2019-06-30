
https://backreference.org/2010/03/26/tuntap-interface-tutorial/

## Tun/tap interfaces

- Tun/tap interfaces allow userspace programs to see raw network traffic.

- Tun/tap interfaces are software-only interfaces, meaning that they exist only in the kernel

- unlike regular network interfaces, they have no physical hardware component

- You can think of a tun/tap interface as a regular network interface that, when the kernel decides that the moment has come to send data "on the wire", instead sends data to some userspace program that is attached to the interface

- When the program attaches to the tun/tap interface, it gets a special file descriptor, reading from which gives it the data that the interface is sending out. 

- the program can write to this special descriptor, and the data  will appear as input to the tun/tap interface. 

- The difference between a tap interface and a tun interface is that a tap interface outputs (and must be given) full ethernet frames, while a tun interface outputs (and must be given) raw IP packets (and no ethernet headers are added by the kernel). 

- Whether an interface functions like a tun interface or like a tap interface is specified with a flag when the interface is created.

- The interface can be transient, meaning that it's created, used and destroyed by the same program; when the program terminates, even if it doesn't explicitly destroy the interface, the interfaces ceases to exist.

- Another option is to make the interface persistent; in this case, it is created using a dedicated utility (like tunctl or openvpn --mktun), and then normal programs can attach to it

- Once a tun/tap interface is in place, it can be used just like any other interface, meaning that IP addresses can be assigned, its traffic can be analyzed, firewall rules can be created, routes pointing to it can be established, etc.

- Program to create or attach to persistent interface need to run as root or with CAP_NET_ADMIN

- The device /dev/net/tun must be opened read/write. That device is also called the clone device.

- call ioctl() system call, with file descriptor obtained in the previous step, the TUNSETIFF constant, and a pointer to a data structure containing the parameters describing the virtual interface - its name and the desired operating mode - tun or tap

- the name of the virtual interface can be left unspecified, in which case the kernel will pick a name by trying to allocate the "next" device of that kind (for example, if tap2 already exists, the kernel will try to allocate tap3, and so on).

- If the ioctl() succeeds, the virtual interface is created and the file descriptor we had is now associated to it, and can be used to communicate.

- At this point, two things can happen. The program can start using the interface right away (probably configuring it with at least an IP address before), and, when it's done, terminate and destroy the interface.

- The other option is to issue a couple of other special ioctl() calls to make the interface persistent

-  optionally set the ownership of the virtual interface to a non-root user and/or group, so programs running as non-root but with the appropriate privileges can attach to the interface later

```c
#include <linux /if.h>
#include <linux /if_tun.h>

int tun_alloc(char *dev, int flags) {

  struct ifreq ifr;
  int fd, err;
  char *clonedev = "/dev/net/tun";

  /* Arguments taken by the function:
   *
   * char *dev: the name of an interface (or '\0'). MUST have enough
   *   space to hold the interface name if '\0' is passed
   * int flags: interface flags (eg, IFF_TUN etc.)
   */

   /* open the clone device */
   if( (fd = open(clonedev, O_RDWR)) < 0 ) {
     return fd;
   }

   /* preparation of the struct ifr, of type "struct ifreq" */
   memset(&ifr, 0, sizeof(ifr));

   ifr.ifr_flags = flags;   /* IFF_TUN or IFF_TAP, plus maybe IFF_NO_PI */

   if (*dev) {
     /* if a device name was specified, put it in the structure; otherwise,
      * the kernel will try to allocate the "next" device of the
      * specified type */
     strncpy(ifr.ifr_name, dev, IFNAMSIZ);
   }

   /* try to create the device */
   if( (err = ioctl(fd, TUNSETIFF, (void *) &ifr)) < 0 ) {
     close(fd);
     return err;
   }

  /* if the operation was successful, write back the name of the
   * interface to the variable "dev", so the caller can know
   * it. Note that the caller MUST reserve space in *dev (see calling
   * code below) */
  strcpy(dev, ifr.ifr_name);

  /* this is the special file descriptor that the caller will use to talk
   * with the virtual interface */
  return fd;
}
```

- char *dev contains the name of an interface (for example, tap0, tun2, etc.). 

- If *dev is '\0', the kernel will try to create the "first" available interface of the requested type

- int flags contains the flags that tell the kernel which kind of interface we want (tun or tap). 

-  IFF_TUN to indicate a TUN device (no ethernet headers in the packets)

- IFF_TAP to indicate a TAP device (with ethernet headers in packets)

A program can use the following code to create a device:

```c
  char tun_name[IFNAMSIZ];
  char tap_name[IFNAMSIZ];
  char *a_name;

  ...

  strcpy(tun_name, "tun1");
  tunfd = tun_alloc(tun_name, IFF_TUN);  /* tun interface */

  strcpy(tap_name, "tap44");
  tapfd = tun_alloc(tap_name, IFF_TAP);  /* tap interface */

  a_name = malloc(IFNAMSIZ);
  a_name[0]='\0';
  tapfd = tun_alloc(a_name, IFF_TAP);    /* let the kernel pick a name */
```

```
    if(owner != -1) {
      if(ioctl(tap_fd, TUNSETOWNER, owner) < 0){
        perror("TUNSETOWNER");
        exit(1);
      }
    }
    if(group != -1) {
      if(ioctl(tap_fd, TUNSETGROUP, group) < 0){
        perror("TUNSETGROUP");
        exit(1);
      }
    }

    if(ioctl(tap_fd, TUNSETPERSIST, 1) < 0){
      perror("enabling TUNSETPERSIST");
      exit(1);
    }
```

- set  persistent and assign ownership to a specific user/group.

```
# openvpn --mktun --dev tun2 --user waldner
TUN/TAP device tun2 opened
Persist state set to: ON

# ip link set tun2 up
# ip addr add 10.0.0.1/24 dev tun2
```

```
# tshark -i tun2

# ping 10.0.0.1

```

- kernel itself is replying to these pings. There is no traffic going through the interface.

- assignment of a /24 IP address to an interface creates a connected route for the whole range through the interface,

```
# ping 10.0.0.2
```

- The kernel sees that the address does not belong to a local interface, and a route for 10.0.0.0/24 exists through the tun2 interface. So it duly sends the packets out tun2. 

-  Note the different behavior here between tun and tap interfaces: with a tun interface, the kernel sends out the IP packet (raw, no other headers are present - try analyzing it with tshark or wireshark), while with a tap interface, being ethernet, the kernel would try to ARP for the target IP address:

- Note that the MAC address for a tap interface is autogenerated by the kernel at interface creation time, but can be changed using the SIOCSIFHWADDR ioctl() 

- being an ethernet interface, the MTU is set to 1500:

```
# ip link show dev tap2
7: tap2:  mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 500
    link/ether 82:03:d4:07:62:b6 brd ff:ff:ff:ff:ff:ff
```

- so far no program is attached to the interface, so all these outgoing packets are just lost. 

- write a program that attaches to a tun interface and reads packets that the kernel sends out that interface.

-  you can run the program as a normal user if the interface is persistent, provided that you have the necessary permissions on the clone device /dev/net/tun, you are the owner of the interface, and select the right mode (tun or tap) for the interface.

```c
 char tun_name[IFNAMSIZ];
  
  /* Connect to the device */
  strcpy(tun_name, "tun77");
  tun_fd = tun_alloc(tun_name, IFF_TUN | IFF_NO_PI);  /* tun interface */

  if(tun_fd < 0){
    perror("Allocating interface");
    exit(1);
  }

  /* Now read data coming from the kernel */
  while(1) {
    /* Note that "buffer" should be at least the MTU size of the interface, eg 1500 bytes */
    nread = read(tun_fd,buffer,sizeof(buffer));
    if(nread < 0) {
      perror("Reading from interface");
      close(tun_fd);
      exit(1);
    }

    /* Do whatever with the data */
    printf("Read %d bytes from device %s\n", nread, tun_name);
  }

  ...
```

```
# ping 10.0.0.2
...

# on another console
$ ./tunclient
Read 84 bytes from device tun77
Read 84 bytes from device tun77
...
```
84 bytes - 20 are for the IP header, 8 for the ICMP header, and 56 are the payload of the ICMP echo message 

- We could analyze the received packet, extract the information needed to reply from the IP header, ICMP header and payload, build an IP packet containing an appropriate ICMP echo reply message, and send it back

-  If using tap, to correctly build reply frames you would probably need to implement ARP in your code.

- Newer virtualization platforms like libvirt use tap interfaces extensively to communicate with guests that support them like qemu/kvm; the interfaces have usually names like vnet0, vnet1 etc. and last only as long as the guest they connect to is running, so they're not persistent, but you can see them if you run ip link show and/or brctl show while guests are running.

- drivers/net/tun.c, functions tun_get_user() and tun_put_user() 

## Tunnels

- we can write a program to just relay the raw data back and forth to a remote host running the same program

```
...
  /* net_fd is the network file descriptor (to the peer), tap_fd is the
     descriptor connected to the tun/tap interface */

  /* use select() to handle two descriptors at once */
  maxfd = (tap_fd > net_fd)?tap_fd:net_fd;

  while(1) {
    int ret;
    fd_set rd_set;

    FD_ZERO(&rd_set);
    FD_SET(tap_fd, &rd_set); FD_SET(net_fd, &rd_set);

    ret = select(maxfd + 1, &rd_set, NULL, NULL, NULL);

    if (ret < 0 && errno == EINTR) {
      continue;
    }

    if (ret < 0) {
      perror("select()");
      exit(1);
    }

    if(FD_ISSET(tap_fd, &rd_set)) {
      /* data from tun/tap: just read it and write it to the network */

      nread = cread(tap_fd, buffer, BUFSIZE);

      /* write length + packet */
      plength = htons(nread);
      nwrite = cwrite(net_fd, (char *)&plength, sizeof(plength));
      nwrite = cwrite(net_fd, buffer, nread);
    }

    if(FD_ISSET(net_fd, &rd_set)) {
      /* data from the network: read it, and write it to the tun/tap interface.
       * We need to read the length first, and then the packet */

      /* Read length */
      nread = read_n(net_fd, (char *)&plength, sizeof(plength));

      /* read packet */
      nread = read_n(net_fd, buffer, ntohs(plength));

      /* now buffer[] contains a full packet or frame, write it into the tun/tap interface */
      nwrite = cwrite(tap_fd, buffer, nread);
    }
  }

...
```





