```javascript
// Define main function (script entry)
// 作为一个小白，为了方便小白写verge-rev脚本，在此提供一个Script脚本模板，方便小白Ctrl+C&V
// 此脚本模板适用于verge-rev的1.7.3以上版本，适用于多设备多人切换订阅使用
// 包含功能：针对筛选的订阅进行分组、规则等字段内容的覆盖以及prepend/append
// 以下的addConfig函数、prepend函数、append函数，小白请勿修改，否则会导致脚本无法正常运行，大佬请自便
const addConfig = (name, config, filename) => {
    if (name.test(filename)) {
        for (let key in Sub[name]) config[key] = Sub[name][key];
        for (let i in Extra[name].preExtra) prepend(config, ...Extra[name].preExtra[i]);
        for (let j in Extra[name].appExtra) append(config, ...Extra[name].appExtra[j]);
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

// Sub部分是实现对dns, proxy-groups, rules等订阅配置文件中所规定字段的内容进行覆盖，其他规定字段也适用
const Sub = {
    // 在此处填入'正则表达式': {}，用于匹配相应的配置，这个正则表达也可以直接写订阅名(就是直接在verge-rev的订阅列表中看到的名称)
    // 注意：别丢引号！防止出错！多个配置之间用<英文的逗号>隔开
    'xx机场': {},
    // 以下配置为一个示例，可以自行参考
    '/^((?!meta.yaml).)*$/i': {// 此处正则表达式意为：匹配所有名不为meta.yaml的文件，有需要可以自行学习
        // 以下是需要覆盖的dns配置，有需求的话可以自行参考官方文档修改，不需要的把这一项全部删除即可
        'dns': {
            // 这里留空会清空dns配置，以下内容为示例
            "enabled": true,
            "listen": "0.0.0.0:1053",
            "ipv6": true,
            "default-nameserver": ["8.8.8.8", "223.5.5.5", "114.114.114.114"],
            "enhanced-mode": "fake-ip",
            "fake-ip-range": "198.18.0.1/16",
            "fake-ip-filter": ["*.lan"],
            "nameserver": ["8.8.8.8", "223.5.5.5", "114.114.114.114", "https://doh.pub/dns-query", "https://dns.alidns.com/dns-query"],
            "fallback-filter": {
                "geoip": true,
                "ipcidr": ["240.0.0.0/4", "0.0.0.0/32"]
            }
        },
        // 以下是需要覆盖的分组配置，有需求的可以自行参考官方文档修改，不需要的把这一项全部删除即可
        'proxy-groups': [
            {
                "name": "🚀 节点选择",
                "type": "select",
                "include-all-proxies": true,
                "proxies": ["♻️ 自动选择", "DIRECT"]
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
        ],
        // 以下是需要覆盖的规则配置，有需求的可以自行参考官方文档修改，不需要的把这一项全部删除即可
        // 注意规则、分组、规则集的对应
        'rules': ["RULE-SET,ProxyLite,🚀 节点选择", "GEOIP,CN,🎯 全球直连", "MATCH,🐟 漏网之鱼"],
        // 以下是需要覆盖的规则集配置，有需求的可以自行参考官方文档修改，不需要的把这一项全部删除即可
        'rule-providers': {
            "LocalAreaNetwork": {
                "type": "http",
                "behavior": "classical",
                "interval": 86400,
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/LocalAreaNetwork.yaml",
                "path": "./profiles/ACL4SSR/LocalAreaNetwork.yaml"
            },
            "UnBan": {
                "type": "http",
                "behavior": "classical",
                "interval": 86400,
                "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/UnBan.yaml",
                "path": "./profiles/ACL4SSR/UnBan.yaml"
            }
        },
    },
};

// Extra部分是对rules、proxies、rule-providers规定字段实现prepend/append功能，不适用其他规定字段
const Extra = {
    '/^((?!meta.yaml).)*$/i': {
        preExtra: [
            ['rules',
                [
                    // 此处填写添加的规则，直接写字符串，多个规则用英文逗号隔开，不需要可以留空
                    "RULE-SET,ProxyLite,🚀 节点选择",
                    "GEOIP,CN,🎯 全球直连", "MATCH,🐟 漏网之鱼",
                ]
            ],
            ['proxies',
                [
                    // 此处填写添加的节点，注意是每个节点都是{...}格式，不是[]格式，多个节点用英文逗号隔开，不需要可以留空
                    {
                        "name": "真的是你呀",
                        "type": "trojan",
                        "server": "0.0.0.0",
                        "port": 443,
                        "password": "00000000000",
                        "network": "ws",
                        "udp": true,
                        "sni": "0.0.0.0",
                        "skip-cert-verify": true
                    },
                ]
            ],
            ['rule-providers',
                {
                    // 此处填写添加的规则集，注意是每个规则集格式：名字:{...}，多个规则集用英文逗号隔开，不需要可以留空
                    "LocalAreaNetwork": {
                        "type": "http",
                        "behavior": "classical",
                        "interval": 86400,
                        "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/LocalAreaNetwork.yaml",
                        "path": "./profiles/ACL4SSR/LocalAreaNetwork.yaml"
                    },
                    "UnBan": {
                        "type": "http",
                        "behavior": "classical",
                        "interval": 86400,
                        "url": "https://raw.gitmirror.com/ACL4SSR/ACL4SSR/master/Clash/Providers/UnBan.yaml",
                        "path": "./profiles/ACL4SSR/UnBan.yaml"
                    }
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
    // 格式为 addConfig(正则表达式, config, filename);
    // 只需要修改正则表达式的内容，config和filename不需要修改
    addConfig(/^((?!meta.yaml).)*$/i, config, filename);
    // addConfig(xx机场, config, filename);
    return config;
}
```
