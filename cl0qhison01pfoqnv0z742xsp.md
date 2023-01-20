# Gitlab CI/CD Strange SSH issue on SSH2_MSG_KEX_ECDH_REPLY

## Problem

A few days ago, I faced a strange issue on my pipeline, and I saw a strange error log that didn't mean anything to me:

```bash
Connection closed by UNKNOWN port 65535
```

So, I decided to enable verbose logs on all 3 levels for every ssh command

> The destination server didn't access the public network directly, so I need to use ProxyJump to access the destination server.

```yaml
 script:
    - ssh -vvvJ jumpbox@$JUMPBOX_IP bahfo2683@$DEST_IP "echo 'hello'"
```

I found that the pipeline was stuck at the Kex algorithm: `bash debug1: expecting SSH2_MSG_KEX_ECDH_REPLY debug3: receive packet: type 96 debug2: channel 0: rcvd eof debug2: channel 0: output open -> drain debug2: channel 0: obuf empty debug2: chan_shutdown_write: channel 0: (i0 o1 sock -1 wfd 5 efd -1 [closed]) debug2: channel 0: output drain -> closed debug1: channel 0: FORCE input drain debug2: channel 0: ibuf empty debug2: channel 0: send eof debug3: send packet: type 96 debug2: channel 0: input drain -> closed Connection closed by UNKNOWN port 65535`

## Solution

After several searches with a headache, I've found the solution 🎉 In this [article](https://www.seei.biz/ssh-fails-to-connect-with-debug1-expecting-ssh2_msg_kex_ecdh_reply/) they describe the exact issue that I had, and the answer is to add the Kex algorithm to `/etc/ssh/ssh_config` in the destination server.

So I added `KexAlgorithms curve25519-sha256,ecdh-sha2-nistp521` below `Host *` on both Jumpbox and Destination Server: \`\`\`bash Host \* KexAlgorithms curve25519-sha256,ecdh-sha2-nistp521

I added \`curve25519-sha256\` as well because I see that in the pipeline the ssh client selected this algorithm.

I ran the pipeline again but guess what? it didn't work!

So, I've came to test something, Firstly, I deleted the Kex Algorithms on Jumpbox Server and then tried to connect to the destination server from Jumpbox and I saw that the \`connection time out!\` So it came to my mind that in the pipeline the same thing happend. I need to set Kex Algorithm on the pipeline as well as both servers.

I edited my \`.gitlab-ci.yml\` file and added\`- sed -i -e "s/Host \\\*/&\\nKexAlgorithms curve25519-sha256,ecdh-sha2-nistp521/g" \` line before anything with ssh:

```yaml
before_script:
    - apk add openssh-client rsync
    - sed -i -e "s/Host \*/&\nKexAlgorithms curve25519-sha256,ecdh-sha2-nistp521/g"  /etc/ssh/ssh_config
```

with `sed -i -e "s/Host \*/&\nKexAlgorithms curve25519-sha256,ecdh-sha2-nistp521/g" /etc/ssh/ssh_config` , I added Kex Algorithms to the ssh\_config file and tried again.

This time the pipeline worked successfully and my application is deployed on the destination server.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647247807785/HE-WGdQth.png align="left")

## Conclusion

It's a rare case for ssh, but keep this in mind if you see this error:

```bash
Connection closed by UNKNOWN port 65535
```

Try two things first:

* Enable Verbose log -vvv for debug levels 1,2 and 3
    
* Check at which step the ssh stuck and google that!
    

If the pipeline stuck at `SSH2_MSG_KEX_ECDH_REPLY` then you should set Kex Algorithms on `.gitlab-ci.yml` and `/etc/ssh/ssh_config` of your server.

Most of them are Mac issues, but in my case, it was the Kex Algorithm.

If you are reading this, I hope this article helps you fix your SSH issue.

I am working at [Tatbiqit](https://tatbiqit.com) as a principal engineer, Please check out my [company website](https://tatbiqit.com) :)

Thank you, and happy coding!