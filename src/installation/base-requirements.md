# Base System Requirements

These are the requirements of the live media. Manual installations as well
as rootfs tarballs typically have lower requirements and can be used as long
as you meet the criteria specified [here](/targets).

| Architecture | CPU    | RAM   | Storage | Platform                      |
| ------------ | ------ | ----- | ------- | ----------------------------- |
| ppc64le      | POWER8 | 96MB  | 700MB   | OpenPOWER, IBM OF             |
| ppc64le-musl | POWER8 | 96MB  | 600MB   | OpenPOWER, IBM OF             |
| ppc64        | 970/G5 | 96MB  | 700MB   | OpenPOWER, IBM/Apple OF       |
| ppc64-musl   | 970/G5 | 96MB  | 600MB   | OpenPOWER, IBM/Apple OF       |
| ppc          | G3     | 64MB  | 700MB   | Apple OF                      |
| ppc-musl     | G3     | 64MB  | 600MB   | Apple OF                      |

> Note: Graphical flavors require more resources, depending on the flavor.

Other platforms may also work, but are currently untested.

It is recommended to have a network connection available during installation,
but it is not strictly necessary.
