#### Note
Bash base64 encode and decode

The options you need to know:
-e or –encode - This option is used to encode any data from standard input or from any file.
-d or –decode - This option is used to decode any encoded data from standard input or from any file.
-i, –ignore-garbage - This option is used to ignore non-alphabet character while decoding.
-n or –noerrcheck - By default, base64 checks error while decoding any data. You can use –n or –noerrcheck option to ignore checking at the time of decoding.

##### Example#1: Encoding text data

``````sh
echo  'linuxhint.com' | base64
echo  -n 'username' | base64
echo  -n 'password' | base64

``````
##### Example#2: Decoding text data
command will decode the encoded text, ‘bGludXhoaW50LmNvbQ==‘ and print the original text as output
``````sh
echo 'bGludXhoaW50LmNvbQo=' | base64 --decode
echo 'bGludXhoaW50LmNvbQo=' | base64 -d

kubectl get secret demo-sa-secret -o jsonpath='{.data.token}'| base64 -d > token.txt
kubectl get secret demo-sa-secret -o jsonpath='{.data.ca\.crt}'| base64 -d > ca-demo.key

``````
