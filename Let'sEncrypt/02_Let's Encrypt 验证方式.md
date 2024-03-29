
https://letsencrypt.org/zh-cn/docs/challenge-types/

这个验证 是为了 to prove, whether you are the owner of that domain 

当您从 Let’s Encrypt 获得证书时，我们的服务器会验证您是否使用 ACME 标准定义的验证方式来验证您对证书中域名的控制权。 大多数情况下，验证由 ACME 客户端自动处理，但如果您需要做出一些更复杂的配置决策，那么了解更多有关它们的信息会很有用。 
如果您不确定怎么做，请使用您的客户端的默认设置或使用 HTTP-01。

When you get a certificate from Let’s Encrypt, our servers validate that you control the domain names in that certificate using “challenges,” as defined by the ACME standard. Most of the time, this validation is handled automatically by your ACME client, but if you need to make some more complex configuration decisions, it’s useful to know more about them. If you’re unsure, go with your client’s defaults or with HTTP-01.

If you want to get a certifiicate form let's encrypt, you need to prove that you are the owner of that particular domain 


# 1 HTTP-01 验证

这是当今最常见的验证方式。 Let’s Encrypt 向您的 ACME 客户端提供一个令牌，然后您的 ACME 客户端将在您对 Web 服务器的 `http://<你的域名>/.well-known/acme-challenge/<TOKEN>`（用提供的令牌替换 `<TOKEN>`）路径上放置指定文件。 该文件包含令牌以及帐户密钥的指纹。 一旦您的 ACME 客户端告诉 Let’s Encrypt 文件已准备就绪，Let’s Encrypt 会尝试获取它（可能从多个地点进行多次尝试）。 如果我们的验证机制在您的 Web 服务器上找到了放置于正确地点的正确文件，则该验证被视为成功，您可以继续申请颁发证书。 如果验证检查失败，您将不得不再次使用新证书重新申请。

我们的 HTTP-01 验证最多接受 10 次重定向。 我们只接受目标为“http:”或“https:”且端口为 80 或 443 的重定向。 我们不目标为 IP 地址的重定向。 当被重定向到 HTTPS 链接时，我们不会验证证书是否有效（因为验证的目的是申请有效证书，所以它可能会遇到自签名或过期的证书）。

HTTP-01 验证只能使用 80 端口。 因为允许客户端指定任意端口会降低安全性，所以 ACME 标准已禁止此行为。

优点
- 它可以轻松地自动化进行而不需要关于域名配置的额外知识。
- 它允许托管服务提供商为通过 CNAME 指向它们的域名颁发证书。
- 它适用于现成的 Web 服务器。

缺点
- 如果您的 ISP 封锁了 80 端口，该验证将无法正常工作（这种情况很少见，但一些住宅 ISP 会这么做）。
- Let’s Encrypt 不允许您使用此验证方式来颁发通配符证书。
- 您如果有多个 Web 服务器，则必须确保该文件在所有这些服务器上都可用。


---

Let's Encrypt gives you a token, and you need to expose it on your webserver. 
这个 token 应该被放在 这个 pattern 所指的位置  `http://<your-domain>/.well-known/acme-challenge/<token>`.  但是通常这步 会被 ACME client 所完成



# 2 DNS-01 验证

==Wildcard Zertifikates can only be verified with the next DNS-01 challenge. not by HTTP-01 challenge ==

Let's Encrepty will query your dns  and will found that particular txt record. And it will validate it by dns 

DNS-01 challenge. This challenge asks you to prove that you control the DNS for your domain name by putting a specific value in a TXT record under that domain name
muss put a specific value as a TXT record under that domain name. 比如 ， for `*.devopsbyexample.io`  wildcard domain, you need to create a TXT record for `_acme-challenge.devopsbyexample.io` with a random token that let's encrypt will generate for you 

例子： want to get a wildcard certificate for the `*/devopsbyexample.io` domain.  You need to create a TXT record for `_acme-challenge.devopsbyexample.io ` with a randon token that let's encryt. 就是 这个 token 的 value 值 填入到 这个 txt record 中 

the certifcate has to be renewal ervey 60 days 

此验证方式要求您在该域名下的 TXT 记录中放置特定值来证明您控制域名的 DNS 系统。 该配置比 HTTP-01 略困难，但可以在某些 HTTP-01 不可用的情况下工作。 它还允许您颁发通配符证书。 在 Let’s Encrypt 为您的 ACME 客户端提供令牌后，您的客户端将创建从该令牌和您的帐户密钥派生的 TXT 记录，并将该记录放在 `_acme-challenge.<YOUR_DOMAIN>` 下。 然后 Let’s Encrypt 将向 DNS 系统查询该记录。 如果找到匹配项，您就可以继续颁发证书！

由于颁发和续期的自动化非常重要，只有当您的 DNS 提供商拥有可用于自动更新的 API 时，使用 DNS-01 验证方式才有意义。 我们的社区在[此处提供了此类 DNS 提供商的列表](https://community.letsencrypt.org/t/dns-providers-who-easily-integrate-with-lets-encrypt-dns-validation/86438)。 您的 DNS 提供商与域名注册商（即向您出售域名的公司）可能相同，也可能不同。 如果您想更改 DNS 提供商，只需在注册商处进行一些小的更改， 无需等待域名即将到期。

需要注意的是，如果将具备完整权限的 DNS API 凭据置于网页服务器上，服务器一旦被黑客攻破会造成更为严重的影响。 最佳做法是使用[权限范围受限的 API 凭据](https://www.eff.org/deeplinks/2018/02/technical-deep-dive-securing-automation-acme-dns-challenge-validation)，或在单独的服务器上执行 DNS 验证并自动将证书复制到 Web 服务器上。

由于 Let’s Encrypt 在查找用于 DNS-01 验证的 TXT 记录时遵循 DNS 标准，因此您可以使用 CNAME 记录或 NS 记录将验证工作委派给其他 DNS 区域。 这可以用于[将 `_acme-challenge` 子域名委派](https://www.eff.org/deeplinks/2018/02/technical-deep-dive-securing-automation-acme-dns-challenge-validation)给验证专用的服务器或区域。 如果您的 DNS 提供商更新速度很慢，那么您也可以使用此方法把验证工作委派给更新速度更快的服务器。

大多数 DNS 提供商都有一个“更新时间”，它反映了从更新 DNS 记录到其在所有服务器上都可用所需的时间。 这个时间可能很难测量，因为这些提供商通常也使用[任播](https://en.wikipedia.org/wiki/Anycast)，这意味着多个服务器可以拥有相同的 IP 地址，并且根据您在世界上的位置，您和 Let’s Encrypt 可能会与不同的服务器通信（并获得不同的应答）。 最好的情况是 DNS API 为您提供了自动检查更新是否完成的方法。 如果您的 DNS 提供商没有这样的方法，您只需将客户端配置为等待足够长的时间（通常多达一个小时），以确保在触发验证之前更新已经完全完成。

您可以为同一名称提供多个 TXT 记录。 例如，如果您同时验证通配符和非通配符证书，那么这种情况可能会发生。 但是，您应该确保清理旧的 TXT 记录，因为如果响应大小太大，Let’s Encrypt 将拒绝该记录。

优点：
- 您可以使用此验证方式来颁发包含通配符域名的证书。
- 即使您有多个 Web 服务器，它也能正常工作。

缺点：
- 在 Web 服务器上保留 API 凭据存在风险。
- 您的 DNS 提供商可能不提供 API。
- 您的 DNS API 可能无法提供有关更新时间的信息。




## 2.1 dns-01 challenge 的例子 




