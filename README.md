```javascript
// Define main function (script entry)
// 作为一个小白，为了方便小白写verge-rev脚本，在此提供一个全局Script脚本模板，方便小白Ctrl+C&V
// 使用：复制全部内容到verge-rev的全局扩展脚本中，然后跟据需要修改
// 此脚本模板适用于verge-rev的1.7.3以上版本，适用于多需求、多设备、多人切换使用
// 包含功能：针对使用需求筛选订阅，进行分组、规则等字段内容的覆盖以及对其rules、proxies、rule-providers进行prepend/append
// 以下的addConfig函数、prepend函数、append函数，小白请勿修改，否则会导致脚本无法正常运行，大佬请自便
const addConfig = (name, usage, config, filename) => {
    if (name.test(filename)) {
        for (let key in Sub[usage]) config[key] = Sub[usage][key];
        for (let i in Extra[usage].preExtra) prepend(config, ...Extra[usage].preExtra[i]);
        for (let j in Extra[usage].appExtra) append(config, ...Extra[usage].appExtra[j]);
    }
}

const prepend = (config, type, content) => {
    if (Array.isArray(content) && content.length > 0) {
        if (type == 'proxies') config.proxies.unshift(...content);
        if (type == 'proxy-groups') config['proxy-groups'].unshift(...content);
        if (type == 'rules') config.rules.unshift(...content);
    }
}

const append = (config, type, content) => {
    if (Array.isArray(content) && content.length > 0) {
        if (type == 'proxies') config.proxies.push(...content);
        if (type == 'proxy-groups') config['proxy-groups'].push(...content);
        if (type == 'rules') config.rules.push(...content);
    }
}

// 国内DNS服务器
const domesticNameservers = [
    "https://dns.alidns.com/dns-query", // 阿里云公共DNS
    "https://doh.pub/dns-query", // 腾讯DNSPod
    "https://doh.360.cn/dns-query" // 360安全DNS
];
// 国外DNS服务器
const foreignNameservers = [
    "https://1.1.1.1/dns-query", // Cloudflare(主)
    "https://1.0.0.1/dns-query", // Cloudflare(备)
    "https://208.67.222.222/dns-query", // OpenDNS(主)
    "https://208.67.220.220/dns-query", // OpenDNS(备)
    "https://194.242.2.2/dns-query", // Mullvad(主)
    "https://194.242.2.3/dns-query" // Mullvad(备)
];
// Sub部分是实现对dns, proxy-groups, rules等订阅配置文件中所规定字段的内容进行覆盖，其他规定字段也适用
const Sub = {
    // 在此处填入'使用需求': {}，用于匹配相应的配置
    // 注意：别丢引号！防止出错！多个配置之间用<英文的逗号>隔开
    '家用': {},
    // 以下配置为一个示例，可以自行参考
    '公司用': {// 此处正则表达式意为：匹配所有名不为meta.yaml的文件，有需要可以自行学习
        // 以下是需要覆盖的dns配置，有需求的话可以自行参考官方文档修改，不需要的把这一项全部删除即可
        'dns': {
            // 这里留空会清空dns配置，以下内容为示例
            "enable": true,
            "listen": "0.0.0.0:1053",
            "ipv6": true,
            "use-system-hosts": false,
            "cache-algorithm": "arc",
            "enhanced-mode": "fake-ip",
            "fake-ip-range": "198.18.0.1/16",
            "fake-ip-filter": [
                // 本地主机/设备
                "+.lan",
                "+.local",
                // Windows网络出现小地球图标
                "+.msftconnecttest.com",
                "+.msftncsi.com",
                // QQ快速登录检测失败
                "localhost.ptlogin2.qq.com",
                "localhost.sec.qq.com",
                // 微信快速登录检测失败
                "localhost.work.weixin.qq.com"
            ],
            "default-nameserver": ["223.5.5.5", "119.29.29.29", "1.1.1.1", "8.8.8.8"],
            "nameserver": [...domesticNameservers, ...foreignNameservers],
            "proxy-server-nameserver": [...domesticNameservers, ...foreignNameservers],
            "nameserver-policy": {
                "geosite:private,cn,geolocation-cn": domesticNameservers,
                "geosite:google,youtube,telegram,gfw,geolocation-!cn": foreignNameservers
            }
        },
        // 以下是需要覆盖的分组配置，有需求的可以自行参考官方文档修改，不需要的把这一项全部删除即可
        'proxy-groups': [
            {
                "name": "🚀 节点选择",
                "type": "select",
                "include-all-proxies": true,
                "proxies": [
                    "♻️ 自动选择",
                    "DIRECT"
                ]
            },
            {
                "name": "♻️ 自动选择",
                "type": "url-test",
                "url": "http://www.gstatic.com/generate_204",
                "interval": 300,
                "tolerance": 50,
                "include-all-proxies": true,
                "proxies": []
            },
            {
                "name": "🌍 国外媒体",
                "type": "select",
                "proxies": [
                    "🚀 节点选择",
                    "♻️ 自动选择",
                    "🎯 全球直连"
                ]
            },
            {
                "name": "📲 电报信息",
                "type": "select",
                "proxies": [
                    "🚀 节点选择",
                    "🎯 全球直连"
                ]
            },
            {
                "name": "Ⓜ️ 微软服务",
                "type": "select",
                "proxies": [
                    "🎯 全球直连",
                    "🚀 节点选择"
                ]
            },
            {
                "name": "🍎 苹果服务",
                "type": "select",
                "proxies": [
                    "🎯 全球直连",
                    "🚀 节点选择"
                ]
            },
            {
                "name": "📢 谷歌FCM",
                "type": "select",
                "proxies": [
                    "🚀 节点选择",
                    "🎯 全球直连",
                    "♻️ 自动选择"
                ]
            },
            {
                "name": "🎯 全球直连",
                "type": "select",
                "proxies": [
                    "DIRECT",
                    "🚀 节点选择",
                    "♻️ 自动选择"
                ]
            },
            {
                "name": "🛑 全球拦截",
                "type": "select",
                "proxies": [
                    "REJECT",
                    "DIRECT"
                ]
            },
            {
                "name": "🍃 应用净化",
                "type": "select",
                "proxies": [
                    "REJECT",
                    "DIRECT"
                ]
            },
            {
                "name": "🐟 漏网之鱼",
                "type": "select",
                "proxies": [
                    "🚀 节点选择",
                    "🎯 全球直连",
                    "♻️ 自动选择"
                ]
            }
        ],
        // 以下是需要覆盖的规则配置，有需求的可以自行参考官方文档修改，不需要的把这一项全部删除即可
        // 注意规则、分组、规则集的对应
        'rules': [
            "RULE-SET,PersonalDirect,🎯 全球直连",
            "RULE-SET,PersonalProxy,🚀 节点选择",
            "RULE-SET,LocalAreaNetwork,🎯 全球直连",
            "RULE-SET,UnBan,🎯 全球直连",
            "RULE-SET,GoogleCN,🎯 全球直连",
            "RULE-SET,SteamCN,🎯 全球直连",
            "RULE-SET,ChinaDomain,🎯 全球直连",
            "RULE-SET,ChinaCompanyIp,🎯 全球直连",
            "RULE-SET,BanAD,🛑 全球拦截",
            "RULE-SET,BanProgramAD,🍃 应用净化",
            "RULE-SET,GoogleFCM,📢 谷歌FCM",
            "RULE-SET,Microsoft,Ⓜ️ 微软服务",
            "RULE-SET,Apple,🍎 苹果服务",
            "RULE-SET,Telegram,📲 电报信息",
            "RULE-SET,ProxyMedia,🌍 国外媒体",
            "RULE-SET,ProxyLite,🚀 节点选择",
            "GEOIP,CN,🎯 全球直连",
            "MATCH,🐟 漏网之鱼"
        ],
        // 以下是需要覆盖的规则集配置，有需求的可以自行参考官方文档修改，不需要的把这一项全部删除即可
        'rule-providers': {
            "LocalAreaNetwork": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/LocalAreaNetwork.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/LocalAreaNetwork.yaml"
            },
            "UnBan": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/UnBan.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/UnBan.yaml"
            },
            "GoogleCN": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/Ruleset/GoogleCN.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/GoogleCN.yaml"
            },
            "SteamCN": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/Ruleset/SteamCN.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/SteamCN.yaml"
            },
            "ChinaDomain": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/ChinaDomain.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/ChinaDomain.yaml"
            },
            "ChinaCompanyIp": {
                "type": "http",
                "behavior": "ipcidr",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/ChinaCompanyIp.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/ChinaCompanyIp.yaml"
            },
            "BanAD": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/BanAD.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/BanAD.yaml"
            },
            "BanProgramAD": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/BanProgramAD.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/BanProgramAD.yaml"
            },
            "GoogleFCM": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/Ruleset/GoogleFCM.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/GoogleFCM.yaml"
            },
            "Microsoft": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/Ruleset/Microsoft.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/Microsoft.yaml"
            },
            "Apple": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/Apple.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/Apple.yaml"
            },
            "Telegram": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/Ruleset/Telegram.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/Telegram.yaml"
            },
            "ProxyMedia": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/ProxyMedia.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/ProxyMedia.yaml"
            },
            "ProxyLite": {
                "type": "http",
                "behavior": "classical",
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/ProxyLite.yaml",
                "interval": 86400,
                "path": "./profiles/ACL4SSR/ProxyLite.yaml"
            }
        }
    }
};

// Extra部分是对rules、proxies、rule-providers规定字段实现prepend/append功能，不适用其他规定字段
const Extra = {
    '家用': {},
    '公司用': {
        preExtra: [
            ['rules',
                [
                    // 此处填写添加的规则，直接写字符串，多个规则用英文逗号隔开，不需要可以留空
                    // "RULE-SET,ProxyLite,🚀 节点选择",
                    // "GEOIP,CN,🎯 全球直连", "MATCH,🐟 漏网之鱼",
                ]
            ],
            ['proxies',
                [
                    // 此处填写添加的节点，注意是每个节点都是{...}格式，不是[]格式，多个节点用英文逗号隔开，不需要可以留空
                    // {
                    //     "name": "真的是你呀",
                    //     "type": "trojan",
                    //     "server": "0.0.0.0",
                    //     "port": 443,
                    //     "password": "00000000000",
                    //     "network": "ws",
                    //     "udp": true,
                    //     "sni": "0.0.0.0",
                    //     "skip-cert-verify": true
                    // },
                ]
            ],
            ['rule-providers',
                {
                    // 此处填写添加的规则集，注意是每个规则集格式：名字:{...}，多个规则集用英文逗号隔开，不需要可以留空
                    // "LocalAreaNetwork": {
                    //     "type": "http",
                    //     "behavior": "classical",
                    //     "interval": 86400,
                    //     "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/LocalAreaNetwork.yaml",
                    //     "path": "./profiles/ACL4SSR/LocalAreaNetwork.yaml"
                    // },
                    // "UnBan": {
                    //     "type": "http",
                    //     "behavior": "classical",
                    //     "interval": 86400,
                    //     "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/UnBan.yaml",
                    //     "path": "./profiles/ACL4SSR/UnBan.yaml"
                    // }
                }
            ]
        ],
        // 此处内容与上面相同，不做多余表述
        appExtra: [
            ['rules',
                [
                    // 此处填写添加的规则，直接写字符串，多个规则用英文逗号隔开，不需要可以留空
                ]
            ],
            ['proxies',
                [
                    // 此处填写添加的节点，注意是每个节点都是{...}格式，不是[]格式，多个节点用英文逗号隔开，不需要可以留空
                ]
            ],
            ['rule-providers',
                {
                    // 此处填写添加的规则集，注意是每个规则集格式：名字:{...}，多个规则集用英文逗号隔开，不需要可以留空
                }
            ]
        ]
    }
};

function main(config, filename) {
    // 在上面配置完毕，就可以在此处调用addConfig函数添加节点、规则、规则集等
    // 格式为 addConfig(正则表达式, 使用需求, config, filename);
    // 只需要修改正则表达式和使用需求的内容，config和filename不需要修改
    // 注意正则表达式的2个斜杠不能丢，具体使用方法可以自行学习
    addConfig(/^.*$/, '家用', config, filename); // /^.*$/表示筛选所有订阅，'家用'表示使用需求对应上文的配置，config和filename保持原样

    // 如果有多个使用需求，可以继续添加
    return config;
}
```
