# IOS 获取IP 的信息

需要获取 当前所有网关，以及其对应的 ip，mac，netmask\(子网掩码\)



`- (void)getAllIPAddresses {`



` struct ifaddrs *interfaces;`

` if(!getifaddrs(&interfaces)) {`

` // Loop through linked list of interfaces`

` struct ifaddrs *interface;`

` for(interface=interfaces; interface; interface=interface->ifa_next) {`

` if(!(interface->ifa_flags & IFF_UP) /* || (interface->ifa_flags & IFF_LOOPBACK) */ ) {`

` continue; // deeply nested code harder to read`

` }`

` const struct sockaddr_in *addr = (const struct sockaddr_in*)interface->ifa_addr;`

` char addrBuf[ MAX(INET_ADDRSTRLEN, INET6_ADDRSTRLEN) ];`

` if(addr && (addr->sin_family==AF_INET)) {`

` NSString *name = [NSString stringWithUTF8String:interface->ifa_name];`

` NSString *macaddr;`

` NSString *type;`

` NSString *mask;`

` NSString *ip;`

` int typeflag = -1;`

` if(addr->sin_family == AF_INET) {`

` if(inet_ntop(AF_INET, &addr->sin_addr, addrBuf, INET_ADDRSTRLEN)) {`

` ip = [NSString stringWithUTF8String:addrBuf];`

` type = @"ipv4";`

` }`



` mask = [NSString stringWithUTF8String:inet_ntoa(((struct sockaddr_in *)interface->ifa_netmask)->sin_addr)];`

` macaddr = [self getMacAddressWithBasName:name];`

` }else {`

` continue;`

` }`







` NSLog(@"name :%@ \n mac:%@ \n mask:%@ \n ip:%@ ",name,macaddr,mask,ip);`





` }`

` }`

` // Free memory`

` freeifaddrs(interfaces);`

` }`

`}`

```//这个根据网关名 获取其ip`

`- (NSString *)getMacAddressWithBasName:(NSString *)badName {`

` int mib[6];`

` size_t len;`

` char *buf;`

` unsigned char *ptr;`

` struct if_msghdr *ifm;`

` struct sockaddr_dl *sdl;`



` mib[0] = CTL_NET;`

` mib[1] = AF_ROUTE;`

` mib[2] = 0;`

` mib[3] = AF_LINK;`

` mib[4] = NET_RT_IFLIST;`



` if ((mib[5] = if_nametoindex([badName UTF8String])) == 0) {`

` printf("Error: if_nametoindex error/n");`

` return NULL;`

` }`



` if (sysctl(mib, 6, NULL, &len, NULL, 0) < 0) {`

` printf("Error: sysctl, take 1/n");`

` return NULL;`

` }`



` if ((buf = (char *)malloc(len)) == NULL) {`

` printf("Could not allocate memory. error!/n");`

` return NULL;`

` }`



` if (sysctl(mib, 6, buf, &len, NULL, 0) < 0) {`

` printf("Error: sysctl, take 2");`

` return NULL;`

` }`



` ifm = (struct if_msghdr *)buf;`

` sdl = (struct sockaddr_dl *)(ifm + 1);`

` ptr = (unsigned char *)LLADDR(sdl);`



` NSString *outstring = [NSString stringWithFormat:@"%02x.%02x.%02x.%02x.%02x.%02x", *ptr, *(ptr+1), *(ptr+2), *(ptr+3), *(ptr+4), *(ptr+5)];`

` free(buf);`



` return [outstring uppercaseString];`

`}`



