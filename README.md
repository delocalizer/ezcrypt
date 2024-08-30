**Alice** wants **Bob** to send her an encrypted directory of files. **Bob** may
be working on Windows, Mac, or Linux, and may not be proficient with the
command line.

## Why this?

`GnuPG` is purpose-built for this use-case, and really good at what it does.
I'm also not a security specialist, so... why roll-my-own?

Two motivations, and one justification:
1. `openssl` is available by default on more environments than `GnuPG` (Linux, macOS, git-bash).
2. `GnuPG` has lots of options which can be confusing for beginners.
3. I think I understand enough to handle this simple case (famous last words!)


Preconditions:
1. **Alice** and **Bob** have `openssl` installed
1. **Bob** has the `encrypt` script as an executable file somewhere on his path
2. **Alice** has the `decrypt` script as an executable file somewhere on her path

For example:
```
git clone https://github.com/delocalizer/ezcrypt
cd ezcrypt
mkdir -p ~/bin
for script in encrypt decrypt; do
    chmod +x ${script}
    ln -s ~/bin ${script}
done
```

## 1. **Alice**: generates an RSA key pair

```
# generate the RSA private key (implicitly contains the public key)
openssl genpkey \
    -algorithm RSA \
    -outform PEM \
    -pkeyopt rsa_keygen_bits:4096 \
    -out private_key.pem \
    # optional: -pass pass:some-passphrase-here
# extract public key:
openssl rsa \
    -pubout \
    -in private_key.pem \
    -out public_key.pem
```

**Alice** shares `public_key.pem` with **Bob** over an unsecured channel.

## 2. **Bob**: encrypts the data

```
encrypt path/to/folder public_key.pem
```

**Bob** shares output `folder.<stamp>.enc` and `datakey.<stamp>.enc` with **Alice** over an unsecured channel.

## 3. **Alice**: decrypts the data

```
decrypt path/to/folder.<stamp>.enc path/to/datakey.<stamp>.enc private_key.pem
```
