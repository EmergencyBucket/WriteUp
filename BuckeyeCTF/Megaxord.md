'''py
with open('megaxord.txt', 'rb') as a:
    megaxord = a.read()


for i in range(0x01,0xFF):
    unencrypted = []
    for j in megaxord:
        unencrypted.append(int(j)^i)
    plain = ""
    for j in unencrypted:
        plain = plain + chr(j)
    if plain.find("buckeye") != -1:
        print(plain)
        print(plain[plain.find("buckeye"):plain.find("}", plain.find("buckeye"))+1])
        break'''
