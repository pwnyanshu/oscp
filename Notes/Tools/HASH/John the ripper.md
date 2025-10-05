hash identifier - wget https://gitlab.com/kalilinux/packages/hash-identifier/-/raw/kali/master/hash-id.py

![[Pasted image 20250819085404.png]]

## Example

It says SHA 1
![[Pasted image 20250819085630.png]]
![[Pasted image 20250819085731.png]]
(gets decrypted with sha1-axcrypt)

Note - 
1. if you dont know the format check it by hash identifier.
2. John may give more than 1 hashing algo, use all of them to find the password
3. enjoy




## Windows auth hash

![[Pasted image 20250819114829.png]]
![[Pasted image 20250819114901.png]]



## Linux shadow unhashing

![[Pasted image 20250819124206.png]]

![[Pasted image 20250819165604.png]]

linux passwords can also be attacked the same way


## Single crack mode (word mangling)
![[Pasted image 20250819175550.png]]


## ssh id_rsa cracking
![[Pasted image 20250819191202.png]]
Note
1. John can just do things on id_rsa
2. Id_rsa is to be converted to jon using ssh2john
3. rest same

## RAR password hash cracking
![[Pasted image 20250819192501.png]]


## zip password cracking
![[Pasted image 20250819193826.png]]