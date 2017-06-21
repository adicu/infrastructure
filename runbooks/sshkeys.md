# How to give people SSH access to the server

First, get a copy of the person's SSH key. Usually, this will live in their
home directory at `~/ssh/id_rsa.pub`, although it might vary.

Make sure that their key ends with either `_rsa.pub` or `_ed25519.pub`. Those
are the only two cryptography algorithms that we currently support (DSA is very
buggy while there are a variety of political concerns over ECDSA).

Their SSH key should be a single line that looks like:

> ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3Y+6WpzH60H3kY+4CDGVUuf1kSesB2McLYfYtj8V+MKADx89bauxLtC0iIj0/5oFhqh/6Y07WVXPdonT//SQ6WymeMirr9KanUCmJJLBxj2yOFxjmBrK65D7A+4TVoLb9vK0kU7ULa85HCPUshsN5cp4AneLPtkIJvt7UBXTqowk6ScQeF3HHwqpR1L56SN5K9J0fpGSQ5Ju8k/CS6eGVCtZsucyotF2uy6fbErYVii1BAeQ7mGApIu2E925pDXd/QibYnOiNO5PlOwxkLb4KQ3KYrvufmH3xFmBWsKwFZuMjZu9FPuTYACChiDb/FIf/z2X+Je1fAPMgeRpa9Bfz alan@Thinkpad-T530

***not*** a multi-line file that looks like:

```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAt2Pulqcx+tB95GPuAgxlVLn9ZEnrAdjHC2H2LY/FfjCgA8fP
W2rsS7QtIiI9P+aBYaof+mNO1lVz3aJ0//0kOlspnjIq6/Smp1ApiSSwcY9sjhcY
5gayuuQ+wPuE1aC2/bytJFO1C2vORwj1LIbDeXKeAJ3iz7ZCCb7e1AV06qMJOknE
Hhdxx8KqUdS+ekjeSvSdH6RkkOSbvJPwkunhlQrWbLnMqLRdrsun2xK2FYotQQHk
O5hgKSLthPduaQ13f0Im2JzojTuT5TsMZC2+CkNymK77n5h98RZgVrCsBWbjI2bv
RT7k2AAgoYg2/xSH/89l/iXtXwDzIHkaWvQX8wIDAQABAoIBAHJRXAgTf0dfMirt
1A741S3Ept0eat2C2UkSLthl9/FatFTG+E5/T389eKj/ePjdYqeT2k1GHH3lVM3D
GHX+wdeSvlW75h+iKUTA5rNz6H1Rr5S/dyjk4gM4hpnb8AkPHyL6u1+awo+1Cygi
wFqaQz3woee2hA2BCpdyoQq/wAsGuYSHHQ+lduWzu0p4q5J8r8jVyvVxpcMpp9r5
WrU10dNkVMWIVG6TX3SqsJACdMCNXBvxtfBuPPrCJJCpBT4fFn1z57+bFYHz3Gp6
jgg8YlrS8rPXyfOXPufleXDUsy72VK39/Cipsk2DhkEkzHVQCe14w/ZMOGlUkfhG
OJ+OZ4ECgYEA646HzXeNicbTnLCq9d+t2ejcS8n4Xm+VuPJ1DvlJ2lycLzdt9St2
lVwFtJj96ccdrx5ADVe8T+QIpfnUZaZ0By9FoLS6RFLC2CvyljWqytkmzUHXOIgh
RzRF9ug7aQKGNk1XSZpm9v1xMXAhYuqDfabxvSrxWjzCKlVgLub0S7MCgYEAx05l
uAUJ3LRPmwKWMqLc7NMa+Dzo5OgxnzC0v+8B4jQjh+iVLWOjX2lZnaSBymBR4a5x
JE9ogLuEeT5J57XezB4dvaHHGaDsqKjKR2Elvx30Jjb2dL9D6glK8tapH0Udv9pC
/inJ2uWLW5JpN+paajtfOfqTaWvecxbU3IDB4sECgYBzyfk5Z7Ycbq7gi/tNp2kW
/58iZiJ/kUxAwHYIKURDYVio4Q9c/8NnwfdQAhB2VRljVnRX2rPHdalGpRrh6MOK
MJOCXrRdF22Nw3SYn8LXuYyYQvAfatMo5CosJ5XklYgRs0zf8lUAvi5hBeRzciG2
p1SXDz/agplTI+qGw6J8fwKBgDzmBoyw9W97pOtPYgd83hZ69r2tFtiC3k6u+ju/
UwsENWsctSBWVqAbt6dEkef9gGd9/tJCdUMIiRTm5HwphTdHaHz+BrEdC9MJKC2h
UIBSLbzThIDtxFmplz4WOzzzyIBLt7ajnCsHgoprdT0BnbjiBVnY59wJesId0tLB
gPzBAoGBAIhGkKvBKESs3TgyOXM3OlONTrd1uJF27WJf5RZKkSwiD5AqRsSX+Km7
E2kZd67xOLZnE4055Wd6svwzKz5XMaRooU4xRcALuU2+iuqprzpKY2X5alCW4zy0
JclufT/FdL3tx5DYNhyONu6xlHxeS7511P+oDjOKtAfTNL2SejGp
-----END RSA PRIVATE KEY-----
```

Make sure that their SSH key has a good personal identifier attached (e.g. the
`alan@Thinkpad-T530` from above) -- otherwise, we won't know who's key it is. Then, make
sure that the key is strong enough by running

```
$ ssh-keygen -lf id_rsa
2048 bc:4f:46:2b:3d:f1:e2:0f:ac:40:99:49:ed:c9:81:a2 alan@Thinkpad-T530 (RSA)
^^^^ ^^^^^^^^^                                       ^^^^                ^^^
|__ Size     |__ Fingerprint                             |__ Comment       |__ Type
```

We are following the [NIST 2016](https://www.keylength.com/en/4/) security
guidelines, which calls for at least a 2048 bit key size for RSA (`id_rsa.pub`
keys) or 224 bits for ED25519 (`id_ed25519.pub`) keys.

After confirming that these keys are suitable, ssh onto `adicu.com` and open
`/root/.ssh/authorized_keys`. Make sure that the key hasn't already been added,
and then added the key to the bottom of the file.
