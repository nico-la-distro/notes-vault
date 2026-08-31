![[ethernet_frame.png]]

**length**

| Élément                            |       Taille |
| ---------------------------------- | -----------: |
| Préambule                          |     7 octets |
| SFD                                |      1 octet |
| Adresse destination + source (MAC) | 6 + 6 octets |
| Type                               |     2 octets |
| FCS                                |     4 octets |
## Preamble and SFD

![[ethernet_preamble_SFD.png]]

## Destination and Source

![[ethernet_dest_src.png]]

## Type / Length

 ![[ethernet_type_length.png]]

## FCS (Frame Check Sequence)

![[ethernet_FCS.png]]