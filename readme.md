```sql
CREATE TABLE `domains` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `word` varchar(6) NOT NULL DEFAULT '',
  `word_len` int(11) NOT NULL,
  `pinyin` varchar(22) NOT NULL DEFAULT '',
  `pinyin_len` int(11) NOT NULL,
  `available` int(11) NOT NULL,
  `status` int(11) NOT NULL,
  `entry_cnt` int(11) NOT NULL,
  `query_cnt` int(11) NOT NULL,
  `query_status` int(11) NOT NULL,
  `created_at` timestamp NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  `updated_at` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=70480 DEFAULT CHARSET=utf8;

```

```php
<?php 

namespace App\Console\Commands;

use App\Console\Commands\Crawler;
use App\Console\Boot;
use App\Http\ZhiHu;
use Curl;
class CrawlerZhiHu extends Boot{

    protected $signature = 'crawler:zhihu {mutix?}';

    /** @var string [描述] */
    protected $description = 'weibo';

    public function __construct()
    {
        parent::__construct();
    }

    public function handle()
    {
        $this->start();

		ZhiHu::unguard(true);

		$this->grap();

        $this->end();
    }

    public function grap()
    {
    	while (true) {
    		$zhihus = ZhiHu::whereStatus(0)->limit(100)->get();
    		foreach ($zhihus as $zhihu) {

				$craw = new Crawler();
    			$url = $zhihu->url;

    			$craw->get($url)->startFilter();

				$zhihu->title = $craw->filter('h2.zm-item-title')->text();

				$zhihu->status = 1;

				$zhihu->save();

				$this->info($url);

				// ZhiHu::saveData(compact('url','status','title'));


				$craw->filter('a.question_link')->each(function($node){
					$link = $node->attr('href');
					$child_url = 'http://www.zhihu.com'.$link;
					if(!ZhiHu::where('url',$child_url)->first())
						ZhiHu::saveData(['url'=>$child_url]);
				});
    		}
    	}
    	




    }


}

 ?>
 
 ```
