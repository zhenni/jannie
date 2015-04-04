<?php
/**
  * baixiao's Chloe Piano Studio
  * 6.1 音乐回复链接有问题
  * 6.2 增加123的回复,并改为多图文回复,修复音乐回复链接
  * 6.3 添加注释 6.4中没有
  * 6.4 添加考级的资料 链接来自jcube
  */

//引入回复图文的函数文件
require_once 'responseNews.func.inc.php';
require_once 'responseMultiNews.func.inc.php';
  
//define your token
define("TOKEN", "baixiao");
$wechatObj = new wechatCallbackapiTest();
if (isset($_GET['echostr'])) {
    $wechatObj->valid();
}else{
    $wechatObj->responseMsg();
}

class wechatCallbackapiTest
{
    public function valid()
    {
        $echoStr = $_GET["echostr"];

        //valid signature , option
        if($this->checkSignature()){
            echo $echoStr;
            exit;
        }
    }

    public function responseMsg()
    {
        //get post data, May be due to the different environments
        $postStr = $GLOBALS["HTTP_RAW_POST_DATA"];

        //extract post data
        if (!empty($postStr)){
                
                $postObj = simplexml_load_string($postStr, 'SimpleXMLElement', LIBXML_NOCDATA);
                $RX_TYPE = trim($postObj->MsgType);

                switch($RX_TYPE)
                {
                    case "text":
                        $resultStr = $this->handleText($postObj);
                        break;
                    case "event":
                        $resultStr = $this->handleEvent($postObj);
                        break;
					case "voice":
						$resultStr = $this->handleVoice($postObj);
                    default:
                        $resultStr = "Unknow msg type: ".$RX_TYPE;
                        break;
                }
                echo $resultStr;
        }else {
            echo "";
            exit;
        }
    }

	public function handleVoice($postObj){
		$fromUsername = $postObj->FromUserName;
        $toUsername = $postObj->ToUserName;
        $time = time();
        $textTpl = "<xml>
                    <ToUserName><![CDATA[%s]]></ToUserName>
                    <FromUserName><![CDATA[%s]]></FromUserName>
                    <CreateTime>%s</CreateTime>
                    <MsgType><![CDATA[%s]]></MsgType>
                    <Content><![CDATA[%s]]></Content>
                    <FuncFlag>0</FuncFlag>
                    </xml>";             
        
		
		$msgType = "text";
		$contentStr = "感谢您的留言和参与。我们会尽快给您回复的[微笑]"."\n"."Chloe钢琴工作室";
		$resultStr = sprintf($textTpl, $fromUsername, $toUsername, $time, $msgType, $contentStr);
		echo $resultStr;
	}
	
    public function handleText($postObj)
    {
        $fromUsername = $postObj->FromUserName;
        $toUsername = $postObj->ToUserName;
        $keyword = trim($postObj->Content);
        $time = time();
        $textTpl = "<xml>
                    <ToUserName><![CDATA[%s]]></ToUserName>
                    <FromUserName><![CDATA[%s]]></FromUserName>
                    <CreateTime>%s</CreateTime>
                    <MsgType><![CDATA[%s]]></MsgType>
                    <Content><![CDATA[%s]]></Content>
                    <FuncFlag>0</FuncFlag>
                    </xml>";
		$musicTpl = "<xml>
				 <ToUserName><![CDATA[%s]]></ToUserName>
				 <FromUserName><![CDATA[%s]]></FromUserName>
				 <CreateTime>%s</CreateTime>
				 <MsgType><![CDATA[music]]></MsgType>
				 <Music>
				 <Title><![CDATA[%s]]></Title>
				 <Description><![CDATA[%s]]></Description>
				 <MusicUrl><![CDATA[%s]]></MusicUrl>
				 <HQMusicUrl><![CDATA[%s]]></HQMusicUrl>
				 </Music>
				 <FuncFlag>0</FuncFlag>
				 </xml>";

		
		if(!empty( $keyword ))
        {
            $msgType = "text";
            $resType = "text";
			
			//天气
            $str = mb_substr($keyword,-2,2,"UTF-8");
            $str_key = mb_substr($keyword,0,-2,"UTF-8");
			
			$str_flag = mb_substr($keyword, 0, 2, "UTF-8");
			$str_message = mb_substr($keyword, 2, mb_strlen($keyword, "UTF-8")-2, "UTF-8");
			
			if($str == '天气' && !empty($str_key)){
                $data = $this->weather($str_key);
                if(empty($data->weatherinfo)){
                    $contentStr = "抱歉，没有查到\"".$str_key."\"的天气信息！";
                } else {
                    $contentStr = "【".$data->weatherinfo->city."天气预报】\n".$data->weatherinfo->ptime."时发布"."\n\n实时天气\n".$data->weatherinfo->weather." ".$data->weatherinfo->temp1."-".$data->weatherinfo->temp2;
                	
                }
            }
			elseif($str_flag == "音乐"){
                $resType = "music_ok";
				$str_music = mb_substr($keyword, 2, 1, "UTF-8");
      			$str_explode = mb_substr($keyword, 3, 20, "UTF-8");
      			$req_music = explode(" ", $str_explode);
      			//$song = mb_substr($keyword, 1, 220, "UTF-8");
      			$song = $req_music[0];
      			$singer = $req_music[1];

      			if ($str_music == " ")
      			{
                    $url_arr = $this->baiduMusic($song, $singer);
          			if (empty($url_arr))
          			{
                        $resType = "text";
            			$contentStr = "非常抱歉哦，小和尚".
            		  "没有找到这首歌，可以换一首嘛[微笑]";
          			}
          			else
          			{
            			include("wx_tpl.php");
            			
                        $resType = "music-ok";
                           
            			$resultStr = sprintf(
            		    $musicTpl, 
            		    $object->FromUserName,
            		    $object->ToUserName, 
            		    $song,
               			$singer,	
               			$url_arr['url'],
               			$url_arr['durl']
               			);
          			}
        		}
                else
                {
                        $resType = "text";
          				$contentStr = "输入格式不正确哦";
                }
                
			}
			elseif($str_flag == "留言"){
				if(empty($str_message)){
					$contentStr = "请在【留言】后加留言信息或给我们意见和建议O(∩_∩)O~我们会尽快给予回复的。谢谢[微笑]"."\n"."Chloe钢琴工作室";
				}else{
					$contentStr = "谢谢您的留言，我们会尽快给予回复的[微笑]"."\n"."Chloe钢琴工作室";
				}
			}
			
			//其他操作项
			else{
				switch($keyword)
                {
                    case "你好": case "hello": case "hi":
						$contentStr = "Hello!~Welcome to Chloe Piano Studio!"."\n"."\n".
							"回复【留言】加您的留言，我们会尽快给予回复。[得意]"."\n"."\n".
							"回复【考级】查看2014年上海音乐学院考级曲目[愉快]"."\n"."\n".
                            "回复【1】，学琴小贴士"."[微笑]"."\n"."\n".
							"回复【2】，名家大讲堂"."[得意]"."\n"."\n".
							"回复【3】，看趣味视频"."[色]"."\n"."\n".
							"回复【4】，去活动中心"."[呲牙]"."\n"."\n".
							"回复【5】，来空中教室"."[愉快]"."\n"."\n".
							"回复【天气】，如【上海天气】，查看天气预报"."[太阳]"."\n"."\n".
							"回复【翻译】，如【翻译piano】或【翻译钢琴】，查看翻译[OK]"."\n"."\n".
							"返回菜单请回复【?】[微笑]"."\n"."\n".
                            // "回复其他与小黄鸡聊天[呲牙]"."\n"."\n".
							"更多消息请查看名片中的历史消息";
                        break;
                    /*
					case "1":
						$contentStr = "回复【1a】，查看《教你如何正确地选择一位钢琴老师》"."\n".
									  "回复【1b】，查看《常见练琴误区》";
						break;
					case "1a": case "选择":
						$resType = "news";
						$record=array(
							'title' =>'教你如何正确地选择一位钢琴老师',
							'description' =>'随着人们生活水平的不断提高，家长们越来越重视孩子的素质培养，尤其是在音乐方面。让孩子学一门乐器，已成为很多家长的共识...',
							'picUrl' => 'http://e.hiphotos.bdimg.com/album/s%3D1000%3Bq%3D90/sign=bb520604272dd42a5b0905ab330b60c4/a5c27d1ed21b0ef477adcb33dfc451da81cb3e11.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10000020&itemidx=1&sign=d98bf232ce239f0972a327c77f01652f#wechat_redirect'
						);
						break;
					*/
					case "1": case"学琴小贴士":
						$resType = "multinews";
						$record[0]=array(
							'title' =>'教你如何正确地选择一位钢琴老师',
							'description' =>'随着人们生活水平的不断提高，家长们越来越重视孩子的素质培养，尤其是在音乐方面。让孩子学一门乐器，已成为很多家长的共识...',
							'picUrl' => 'http://e.hiphotos.bdimg.com/album/s%3D1000%3Bq%3D90/sign=bb520604272dd42a5b0905ab330b60c4/a5c27d1ed21b0ef477adcb33dfc451da81cb3e11.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10000020&itemidx=1&sign=d98bf232ce239f0972a327c77f01652f#wechat_redirect'
						);
						$record[1]=array(
							'title' =>'常见练琴误区',
							'description' =>'练习是学琴过程中最重要的步骤，练琴的过程就是学生提高和进步的过程，上课只是对这个过程的检验和指导。因此，培养一个良好的练琴习惯显得犹为重要...',
							'picUrl' => 'http://f.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=51b2b19779cb0a4681228f385b53cd55/f9dcd100baa1cd112fca214dbb12c8fcc3ce2d5a.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10013007&itemidx=1&sign=f44dbde0faeb149ebe7ce42bca6c5453#wechat_redirect'
						);
						$record[2]=array(
							'title' =>'谈练琴(一)',
							'description' =>'很多琴童一坐下就马上开弹老师布置的曲子，头脑不清醒，各种节奏音高错乱不堪，基础太薄弱乎！！...',
							'picUrl' => 'http://a.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=12a554614890f60300b098460922886a/9a504fc2d56285357dfa0a7192ef76c6a7ef634c.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10001028&itemidx=1&sign=ac3c9ceda103af3ffa2bdace0b5277d3#wechat_redirect'
						);
						$record[3]=array(
							'title' =>'学习钢琴对孩子整体素质的培养',
							'description' =>'法国作家雨果说过，开启人类智慧大门的三把钥匙是：字母（语文）、数字（数学）、音符（音乐）...',
							'picUrl' => 'http://g.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=9dc30c639045d688a702b6a594f2466f/500fd9f9d72a60597b13244a2a34349b033bba5b.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10000015&itemidx=1&sign=ccc868c4c3925b2c1497195754b23f0f#wechat_redirect'
						);
						$record[4]=array(
							'title' =>'如何学习钢琴（之一）学习钢琴与学习其他兴趣课程的不同之处',
							'description' =>'学习钢琴孩子们的家长圈中总会流传一句话:“读书好的孩子未必能弹好钢琴，但是能弹好钢琴的孩子只要认真学习，成绩必定好！”...',
							'picUrl' => 'http://d.hiphotos.bdimg.com/album/s%3D740%3Bq%3D90/sign=80759d424d4a20a4351e3ec3a069e91f/a9d3fd1f4134970af32dd57f97cad1c8a7865d00.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10000025&itemidx=1&sign=a47665b905ceeefe9a05cd6c4b27868a#wechat_redirect'
						);
						$record[5]=array(
							'title' =>'如何学习钢琴（之二）正确良好的练琴习惯是学好钢琴的关键',
							'description' =>'孩子要想长久的学钢琴，一定需要的是正确的学习方法和学习习惯。孩子每天练琴的习惯，则变成了家长未来能否成功教育孩子的第一课。...',
							'picUrl' => 'http://d.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=e91fc11389d4b31cf43c90bab7e61c0e/f2deb48f8c5494ee53edd5492ff5e0fe98257e97.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10000027&itemidx=1&sign=12f0b75a968648a7eaeb1424e0397105#wechat_redirect'
						);
						$record[6]=array(
							'title' =>'如何学习钢琴（之三）宝贝学习钢琴，家长准备好了吗？',
							'description' =>'学钢琴既可以提高修养、陶冶情操、促进孩子手眼协调和大脑发育，又有助于好习惯的养成、培养毅力锻炼性格，一举多得...',
							'picUrl' => 'http://h.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=64fa472541a98226bcc12f26bab28270/b90e7bec54e736d1a010495899504fc2d562695b.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10000022&itemidx=1&sign=8f4697f478932db09f95a7dd1ff0ae76#wechat_redirect'
						);
						$record[7]=array(
							'title' =>'不懈的努力与明确目标，是学习钢琴能否产生或保持快乐的关键！',
							'description' =>'经常会在网上看到一些家长对于孩子的学琴期望是：“只要孩子开心就好了，弹着玩玩，不要给他太大压力”；“只要孩子有兴趣学钢琴就好了”等...',
							'picUrl' => 'http://a.hiphotos.bdimg.com/album/s%3D740%3Bq%3D90/sign=ea9e3d71708b4710ca2fffc8f3f5b2c0/730e0cf3d7ca7bcb5a79027fbc096b63f624a864.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10001006&itemidx=1&sign=04bf5b9d283040b9a5e8d0594fdcf679#wechat_redirect'
						);
						$record[8]=array(
							'title' =>'如何鼓励孩子学音乐',
							'description' =>'就像所有的好父母一样，你也希望自己的孩子能够拥有最好的生活：得到良好的教育，吃有营养的食物并拥有一定的储蓄。这意味着你的孩子需要具备良好的生活习惯、富有创造力并且懂得享受生活的乐趣...',
							'picUrl' => 'http://d.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=1ce35672cafcc3ceb0c0cd32a275edf9/0d338744ebf81a4c0afc3699d52a6059242da690.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10001004&itemidx=1&sign=0ec89f0dbe3c032e4b86746d7eed40e4#wechat_redirect'
						);
						$record[9]=array(
							'title' =>'父母应该如何帮助孩子在学习音乐的过程中感到愉快而减少挫折感呢？',
							'description' =>'许多家长会提出这样的问题：「我们不懂音乐，不知道要让小孩上音乐班还是学钢琴？」、「我的孩子学钢琴，但是我又不懂音乐，不知道他学得怎样？」...',
							'picUrl' => 'http://a.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=a02ff944a918972ba73a04cbd6fd40f8/342ac65c103853438bcb42719113b07eca80885e.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10001031&itemidx=1&sign=bc9d5573201e9ad70f6d3fbf9bd8ed3f#wechat_redirect'
						);
						/*$record[2]=array(
							'title' =>'',
							'description' =>'...',
							'picUrl' => '',
							'url' =>''
						);*/
						break;
						
					case "2": case"名家大讲堂":
						$resType = "multinews";
						$record[0]=array(
							'title' =>'傅聪：《家书》其实我从来都不看，我不敢看',
							'description' =>'六十年前，他赴波兰留学，他的父亲对他说，“做人第一，其次才是做艺术家，再其次才是做音乐家，最后才是做钢琴家”...',
							'picUrl' => 'http://g.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=e0b4013ef4246b607f0eb675dbc8213d/4610b912c8fcc3ce9d40965e9045d688d53f20de.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10001014&itemidx=1&sign=b602dfa40d403e0e33f813dce1a94fc1#wechat_redirect'
						);
						$record[1]=array(
							'title' =>'琴童家长十忌---鲍慧乔',
							'description' =>'鲍蕙荞是中国最著名的钢琴家之一，从1970年起到现在，她一直任中国交响乐团钢琴独奏家，并获国家一级演奏员称号。现为中国音乐家协会全国器乐演奏（业余）考级委员会专家委员会副主任、中央乐团（中国交响乐团前身）社会音乐学院副院长...',
							'picUrl' => 'http://c.hiphotos.bdimg.com/album/s%3D740%3Bq%3D90/sign=692fc12a3b12b31bc36ccf2db6234747/37d12f2eb9389b505560bd3f8735e5dde7116e3b.jpg',
							'url' =>'http://mp.weixin.qq.com/s?__biz=MjM5MzA1NzI4Nw==&mid=10013142&idx=1&sn=b63c391f0b7cef00832e0311080d71e6#rd'
						);
						$record[2]=array(
							'title' =>'写作业和练琴不打架',
							'description' =>'文/茅为蕙（旅美钢琴家）四五岁学琴，虽然孩子小，但可以全身心投入。而开始上学的琴童，就会碰到现实的问题：学业和琴业如何平衡...',
							'picUrl' => 'http://h.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=913e94e99c510fb37c197396e903f3e4/9213b07eca8065387b247e4395dda144ac348290.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10001025&itemidx=1&sign=69e7c60527be648e978833155fcf2caf#wechat_redirect'
						);
						$record[3]=array(
							'title' =>'学乐器会不会影响孩子的学习成绩',
							'description' =>'学习乐器是不是会影响学习成绩？这也许只有中国的父母有这种忧虑吧...',
							'picUrl' => 'http://h.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=2261247b0cf41bd5de53ecf561eababa/359b033b5bb5c9ea4ab1c0b3d739b6003af3b324.jpg',
							'url' =>'http://mp.weixin.qq.com/s?__biz=MjM5MzA1NzI4Nw==&mid=10013137&idx=1&sn=c271435108c650a608414b1be1021d34#rd'
						);
						$record[4]=array(
							'title' =>'关于“手型”问题的探讨---周广仁教授谈“手型”',
							'description' =>'经常有些家长会问我，我们孩子的手型对不对？或者有些家长说，他们之前的钢琴老师有的强调手型，有的却说手型不重要...',
							'picUrl' => 'http://g.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=8eeae950eb24b899da3c7d395e3626e4/5d6034a85edf8db16dc1bf340b23dd54574e74f7.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10013112&itemidx=1&sign=2a0671351e23a2f78095cf7a6ee9f115#wechat_redirect'
						);
						$record[5]=array(
							'title' =>'【音乐语录】',
							'description' =>'音乐从来没有所谓"专业"或"业余"之分。只有"心中有乐"与"心中无乐"之分。只有"真心愛乐"与"并不愛乐"之分。...',
							'picUrl' => 'http://g.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=973f51e6cb177f3e1434f80c40ff00b6/472309f79052982234f9fa93d5ca7bcb0b46d49c.jpg',
							'url' =>'http://mp.weixin.qq.com/s?__biz=MjM5MzA1NzI4Nw==&mid=10013129&idx=1&sn=d913d5d07a42489a6cc830822b71a78f#rd'
						);
						$record[6]=array(
							'title' =>'送给琴童和家长的几句话',
							'description' =>'琴童们演奏时应注意的"5个度"...',
							'picUrl' => 'http://h.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=f2a2a02e9058d109c0e3adb3e168f7ce/0bd162d9f2d3572c3e295f8c8813632763d0c393.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10013055&itemidx=2&sign=3b0919179decfa4a4e8b70c486e14deb#wechat_redirect'
						);
						$record[7]=array(
							'title' =>'【乐·读】',
							'description' =>'每一个音乐作品其实都在表现一种性格，一份心情，一片场景，一个故事；...',
							'picUrl' => 'http://c.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=baf7ff44a918972ba73a04cbd6fd40f8/342ac65c10385343911344719113b07ecb808886.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10013046&itemidx=1&sign=f21ab59b639905fdbe307004c8f75845#wechat_redirect'
						);
						$record[8]=array(
							'title' =>'如何考级，才能趋利避害',
							'description' =>'我认为最重要的原则，是学生现阶段的程度和考级的曲目应当是相同的或是相近的。...',
							'picUrl' => 'http://f.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=6480ff43df54564ee165e03883eea7f3/03087bf40ad162d9df07a14c13dfa9ec8b13cdf1.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10001040&itemidx=1&sign=7f2a77b875ab6a8520339c385f3a2b66#wechat_redirect'
						);
						$record[9]=array(
							'title' =>'[乐·读]',
							'description' =>'经常听到这样说法：孩子还小先随便找个老师培养一下兴趣...',
							'picUrl' => 'http://c.hiphotos.bdimg.com/album/s%3D740%3Bq%3D90/sign=b56a387b9f16fdfadc6cc4ea84b4fd69/a8773912b31bb051ea9c7078347adab44aede022.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10013004&itemidx=1&sign=393637ad6dd364116ada1fdf4d2d9bed#wechat_redirect'
						);
						break;
						
					case "3": case "趣味视频":
						$resType = "multinews";
						$record[0]=array(
							'title' =>'又有新花样了！来看看十二人共弹一架钢琴，一起接力合奏！',
							'description' =>'来看看这回钢琴又玩儿出什么花样了？来自华盛顿音乐学院的十二位教授接力演奏，共同谱出令人惊艳的乐章！十二双手一起弹奏，场面太欢乐了...',
							'picUrl' => 'http://g.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=b1e3a36072cf3bc7ec00c9ede13081d0/730e0cf3d7ca7bcbdcac9842bc096b63f724a88c.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10012121&itemidx=1&sign=559ca4c0b18f821cacab04f1b810843e#wechat_redirect'
						);
						$record[1]=array(
							'title' =>'不一样的 Rolling In The Deep',
							'description' =>'大家都对Adele的“Rolling In The Deep”记忆深刻，下面我们来看一下用钢琴和大提琴共同演绎的“Rolling In The Deep”，会有什么不一样的味道...',
							'picUrl' => 'http://f.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=8cdefd57324e251fe6f7e0f997b6f266/b3fb43166d224f4aed4e2c7e0bf790529922d18c.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10012150&itemidx=1&sign=309c42b4672d1dd45b1c4206bfc3cf39#wechat_redirect'
						);
						$record[2]=array(
							'title' =>'【音乐趣频欣赏】：这是我见过的最牛的钢琴四手联弹（没有之一）',
							'description' =>'真正的“Play The Piano” 看看这两个人怎样天衣无缝的配合 把钢琴“玩儿”出了新的花样...',
							'picUrl' => 'http://d.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=7ad26b6a324e251fe6f7e0f997b6f266/b3fb43166d224f4a1b42ba430bf790529922d183.jpg',
							'url' =>'http://mp.weixin.qq.com/s?__biz=MjM5MzA1NzI4Nw==&mid=10013271&idx=1&sn=6037665a6fa3f995a4f94ec6391b7fa3#rd'
						);
						$record[3]=array(
							'title' =>'钢琴版“忐忑”',
							'description' =>'神曲“忐忑”在钢琴上也是个超级“神曲” 技巧之难完全可以比拟李斯特“超技练习曲”...',
							'picUrl' => 'http://d.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=3c489b69cb3d70cf48faae0cc8ecea71/9922720e0cf3d7ca38bdf626f01fbe096b63a913.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10013117&itemidx=1&sign=c01a09f0c102b7257e3ae555d3740806#wechat_redirect'
						);
						$record[4]=array(
							'title' =>'音乐趣频：“猫咪”协奏曲',
							'description' =>'来看看这只猫咪与整个乐队天衣无缝的配合吧~ 此猫乃神猫O(∩_∩)O哈哈...',
							'picUrl' => 'http://c.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=9691688ef21f3a295ec8d1cfa9158740/86d6277f9e2f070832fde350eb24b899a801f2e2.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10013107&itemidx=1&sign=644b2fd5708fbd4d2be3c44f59ea72cb#wechat_redirect'
						);
						$record[5]=array(
							'title' =>'音乐欣赏：“加勒比海盗”主题曲 钢琴版',
							'description' =>'在这里“chloe钢琴工作室”为大家奉上一首钢琴版“加勒比海盗”主题曲...',
							'picUrl' => 'http://g.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=8bd219bde51190ef05fb96defe2ba667/b21bb051f8198618389d26d448ed2e738ad4e68c.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10013091&itemidx=1&sign=f1266c9853ef4116f052d0cdd394a1ae#wechat_redirect'
						);
						$record[6]=array(
							'title' =>'音乐趣频：会分辨音高的高智商金毛犬 （ps：这次升级了：从单音、到分解和弦及简单曲调）',
							'description' =>'从单音、分解和弦、再到半音阶和简单曲调，这只金毛狗狗都准确的在键盘上“弹奏”出来了...',
							'picUrl' => 'http://d.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=afea6e25d488d43ff4a995f34d2ee96a/d8f9d72a6059252d87c3d454369b033b5ab5b98e.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10013080&itemidx=1&sign=d460def76146e608dfe5620463524d54#wechat_redirect'
						);
						$record[7]=array(
							'title' =>'音乐趣频欣赏：奥巴马演唱的说唱版“Jingle Bells”',
							'description' =>'大家来听听“Obama”先生的Rap，是不是很“欢型”O(∩_∩)O哈哈...',
							'picUrl' => 'http://f.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=5f7dab4c13dfa9ecf92e521652e0cc72/d058ccbf6c81800ab476405eb33533fa838b47cb.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10013085&itemidx=1&sign=aac69531d8b856afd91824404358d877#wechat_redirect'
						);
						$record[8]=array(
							'title' =>'“猫和老鼠”版 李斯特第二匈牙利狂想曲',
							'description' =>'来重温一下大家小时候都看过的“TOM & JERRY”...',
							'picUrl' => 'http://g.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=f688ad23f9f2b211e02e814ffab05e49/e7cd7b899e510fb3e994bcc7db33c895d0430cd5.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10013030&itemidx=1&sign=5c5ed7639386322b1c35ba5e5824ce8b#wechat_redirect'
						);
						$record[9]=array(
							'title' =>'5岁华裔钢琴神童Ryan Wang演奏轰动全美',
							'description' =>'5岁华裔钢琴神童Ryan Wang演奏轰动全美...',
							'picUrl' => '',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10013055&itemidx=1&sign=9223a6dbadf00ae0afedb4eca8425c45#wechat_redirect'
						);

						break;
					
                    case "游戏":case "遊戲":
  		            	$resType = "multinews";
						$record[0]=array(
							'title' =>'2048小游戏',
							'description' =>'',
							'picUrl' => 'http://www.8090yxs.com/uploads/allimg/140331/1-14033113541TP.png',
							'url' =>'http://1.jannie.sinaapp.com/2048-master/index.html'
						);
						$record[1]=array(
							'title' =>'困住神经猫',
							'description' =>'',
							'picUrl' => 'http://i1.img.969g.com/15666/imgx2014/10/02/320_023550_d2000.jpg',
							'url' =>'http://1.jannie.sinaapp.com/%E7%A5%9E%E7%BB%8F%E7%8C%AB/index.html'
						);
						break;
                    
                    
					case "4": case "活动中心":
						$resType = "multinews";
						$record[0]=array(
							'title' =>'“今夜我登台”音乐会回顾',
							'description' =>'为了给学生们一个展示自我的机会，12月8号晚，chloe钢琴工作室组织了一个小型的钢琴演奏交流会------“今夜我登台”...',
							'picUrl' => 'http://h.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=c4b7e7a3d762853596e0d620a0df4db7/728da9773912b31befb650788418367adbb4e18d.jpg',
							'url' =>'http://mp.weixin.qq.com/mp/appmsg/show?__biz=MjM5MzA1NzI4Nw==&appmsgid=10012106&itemidx=1&sign=fb898fa43e7f2ffa183c3f6f78a9943c#wechat_redirect'
						);
						$record[1]=array(
							'title' =>'钢琴演奏表演小贴士',
							'description' =>'钢琴演奏表演小贴士...',
							'picUrl' => 'http://c.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=46b96b56369b033b2888f8db25fe0da2/0ff41bd5ad6eddc4c349c2a93bdbb6fd52663331.jpg',
							'url' =>'http://mp.weixin.qq.com/s?__biz=MjM5MzA1NzI4Nw==&mid=10013314&idx=1&sn=17c076624b4a5063e38b7a9aec777f82#rd'
						);
						/*$record[1]=array(
							'title' =>'“今夜我登台”之录像篇',
							'description' =>'“今夜我登台”的录像出炉啦！...',
							'picUrl' => 'http://h.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=c4b7e7a3d762853596e0d620a0df4db7/728da9773912b31befb650788418367adbb4e18d.jpg',
							'url' =>'http://mp.weixin.qq.com/s?__biz=MjM5MzA1NzI4Nw==&mid=10013943&idx=1&sn=4f899af281088b55d349f5b9697038f1#rd'
						);
						$record[2]=array(
							'title' =>'2013年12月28日“敬老迎新”音乐会回顾',
							'description' =>'2013年12月28日晚7时15分，在贺绿汀音乐厅举办了“陪长辈听经典”系列音乐会，由上海歌剧院交响乐团一直热心敬老文化事业的青年单簧管演奏家赵超领衔，与特邀青年钢琴家白皛（chloe钢琴工作室创办人）...',
							'picUrl' => 'http://a.hiphotos.bdimg.com/album/s%3D740%3Bq%3D90/sign=fc09587e38292df593c3ae118c0a2d5d/4bed2e738bd4b31cefe609ea85d6277f9f2ff88d.jpg',
							'url' =>'http://mp.weixin.qq.com/s?__biz=MjM5MzA1NzI4Nw==&mid=10013945&idx=1&sn=3f066eb01a80dd193bb8f1429a3ccad8#rd'
						);
						$record[3]=array(
							'title' =>'新春欢乐马上送 -- Chloe钢琴社空中音乐秀',
							'description' =>'马上送好礼~chloe钢琴社送福利啦...',
							'picUrl' => 'http://g.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=001bfa07d2160924d825a61ae4370e8b/3b87e950352ac65ca6defb22f9f2b21192138a8c.jpg',
							'url' =>'http://mp.weixin.qq.com/s?__biz=MjM5MzA1NzI4Nw==&mid=10013854&idx=1&sn=9e42f72ebdaccc84ef928e225c538054#rd'
						);*/
						break;
					
                    case "music":
						$resType = "multinews";
						$record[0]=array(
							'title' =>'音乐链接',
							'description' =>'我们来试一试~',
							'picUrl' => 'http://c.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=baf7ff44a918972ba73a04cbd6fd40f8/342ac65c10385343911344719113b07ecb808886.jpg',
							'url' =>'http://m.mobibao.net/webpage/123'
						);
                    	break;
                    
                    case "论坛":
						$resType = "multinews";
						$record[0]=array(
							'title' =>'Chloe论坛',
							'description' =>'快来参与讨论~',
							'picUrl' => 'http://c.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=baf7ff44a918972ba73a04cbd6fd40f8/342ac65c10385343911344719113b07ecb808886.jpg',
							'url' =>'http://m.mobibao.net/forum/54'
						);
                    	break;
                    
                    case "考级":
						$contentStr = "回复【x级y】查看考x级的第y首曲目的MP3, 或者曲目名称。"."\n"."\n".
							"如【5级1】或【10级3】，每级曲目共有三首"."\n"."\n".
							"返回菜单请回复【?】[微笑]"."\n"."\n";
						break;
					
					/* 链接失效
						$resType = "multinews";
						$record[0]=array(
							'title' =>'2014年上海音乐学院 钢琴考级曲级（1-5级）',
							'description' =>'一眨眼，又到了一年一度的考级时期，本工作室把2014-2015年的上海音乐学院考级曲目整理出来，便于大家随时随听～',
							'picUrl' => 'http://f.hiphotos.bdimg.com/album/s%3D1100%3Bq%3D90/sign=8cdefd57324e251fe6f7e0f997b6f266/b3fb43166d224f4aed4e2c7e0bf790529922d18c.jpg',
							'url' =>'http://m.mobibao.net/webpage/c983783c32'
						);
                    	$record[1]=array(
							'title' =>'2014年上海音乐学院 钢琴考级曲级（6-10级）',
							'description' =>'一眨眼，又到了一年一度的考级时期，本工作室把2014-2015年的上海音乐学院考级曲目整理出来，便于大家随时随听～',
							'picUrl' => 'http://c.hiphotos.bdimg.com/album/s%3D740%3Bq%3D90/sign=b56a387b9f16fdfadc6cc4ea84b4fd69/a8773912b31bb051ea9c7078347adab44aede022.jpg',
							'url' =>'http://m.mobibao.net/webpage/9e31d27f74'
						);
                    	break;
                    */
					
					case "5":
						$contentStr = "Chloe钢琴工作室送福利啦O(∩_∩)O~"."\n".
									  "如果你想得到Chloe钢琴工作室的专业点评，可以用以下方法录制一段音频，并发给我们，我们会在24小时内给于专业的点评和建议噢*^__^*"."\n".
									  "方法如下："."\n".
									  "按住【按住说话】键进行录制，并发送。";
						break;
					
					case "?": case "？":
						$contentStr = "嘿哈，欢迎来到Chloe钢琴工作室，本人一定会认真回复每一个问题和留言。"."\n"."\n".
							"回复【留言】加您的留言，我们会尽快给予回复。[得意]"."\n"."\n".
                            "回复【考级】查看2014年上海音乐学院考级曲目[愉快]"."\n"."\n".
							"回复【1】，学琴小贴士"."[微笑]"."\n"."\n".
							"回复【2】，名家大讲堂"."[得意]"."\n"."\n".
							"回复【3】，看趣味视频"."[色]"."\n"."\n".
							"回复【4】，去活动中心"."[呲牙]"."\n"."\n".
							"回复【5】，来空中教室"."[愉快]"."\n"."\n".
							"回复【天气】，如【上海天气】，查看天气预报"."[太阳]"."\n"."\n".
							"回复【翻译】，如【翻译piano】或【翻译钢琴】，查看翻译[OK]"."\n"."\n".
							"返回菜单请回复【?】[微笑]"."\n"."\n".
                            // "回复其他与小黄鸡聊天[呲牙]"."\n"."\n".
							"更多消息请查看名片中的历史消息";
						break;
					
					case "C小调练习曲": case "c小调练习曲":
						$resType = "music";
						$music_name = "C小调练习曲";
						$music_singer = "肖邦";
						$musicUrl   = "http://202.103.244.172/text/jxzy/music2/ozyy/music/10/10.mp3";
						$HQmusicUrl = "http://202.103.244.172/text/jxzy/music2/ozyy/music/10/10.mp3";
						break;
					
					case "1级1":
						$resType = "music";
						$music_name = "一级曲目1";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/1.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/1.mp3";
						break;
					case "1级2": 
						$resType = "music";
						$music_name = "一级曲目2";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/2.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/2.mp3";
						break;
					case "1级3": 
						$resType = "music";
						$music_name = "一级曲目3";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/3.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/3.mp3";
						break;
					case "2级1": 
						$resType = "music";
						$music_name = "二级曲目1";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/4.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/4.mp3";
						break;
					case "2级2": 
						$resType = "music";
						$music_name = "二级曲目2";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/5.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/5.mp3";
						break;
					case "2级3": 
						$resType = "music";
						$music_name = "二级曲目3";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/6.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/6.mp3";
						break;
					case "3级1": 
						$resType = "music";
						$music_name = "三级曲目1";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/7.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/7.mp3";
						break;
					case "3级2": 
						$resType = "music";
						$music_name = "三级曲目2";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/8.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/8.mp3";
						break;
					case "3级3": 
						$resType = "music";
						$music_name = "三级曲目3";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/9.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/9.mp3";
						break;
					case "4级1": 
						$resType = "music";
						$music_name = "四级曲目1";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/10.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/10.mp3";
						break;
					case "4级2": 
						$resType = "music";
						$music_name = "四级曲目2";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/11.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/11.mp3";
						break;
					case "4级3": 
						$resType = "music";
						$music_name = "四级曲目3";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/12.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/12.mp3";
						break;
					case "5级1": 
						$resType = "music";
						$music_name = "五级曲目1";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/13.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/13.mp3";
						break;
					case "5级2": 
						$resType = "music";
						$music_name = "五级曲目2";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/14.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/14.mp3";
						break;
					case "5级3": 
						$resType = "music";
						$music_name = "五级曲目3";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/15.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/15.mp3";
						break;
					case "6级1": 
						$resType = "music";
						$music_name = "六级曲目1";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/16.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/16.mp3";
						break;
					case "6级2": 
						$resType = "music";
						$music_name = "六级曲目2";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/17.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/17.mp3";
						break;
					case "6级3": 
						$resType = "music";
						$music_name = "六级曲目3";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/18.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/18.mp3";
						break;
					case "7级1": 
						$resType = "music";
						$music_name = "七级曲目1";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/19.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/19.mp3";
						break;
					case "7级2": 
						$resType = "music";
						$music_name = "七级曲目2";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/20.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/20.mp3";
						break;
					case "7级3": 
						$resType = "music";
						$music_name = "七级曲目3";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/21.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/21.mp3";
						break;
					case "8级1": 
						$resType = "music";
						$music_name = "八级曲目1";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/22.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/22.mp3";
						break;
					case "8级2": 
						$resType = "music";
						$music_name = "八级曲目2";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/23.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/23.mp3";
						break;
					case "8级3": 
						$resType = "music";
						$music_name = "八级曲目3";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/24.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/24.mp3";
						break;
					case "9级1": 
						$resType = "music";
						$music_name = "九级曲目1";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/25.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/25.mp3";
						break;
					case "9级2": 
						$resType = "music";
						$music_name = "九级曲目2";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/26.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/26.mp3";
						break;
					case "9级3": 
						$resType = "music";
						$music_name = "九级曲目3";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/27.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/27.mp3";
						break;
					case "10级1": 
						$resType = "music";
						$music_name = "十级曲目1";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/28.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/28.mp3";
						break;
					case "10级2": 
						$resType = "music";
						$music_name = "十级曲目2";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/29.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/29.mp3";
						break;
					case "10级3": 
						$resType = "music";
						$music_name = "十级曲目3";
						$music_singer = "Chloe钢琴工作室";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/34364/30.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/34364/30.mp3";
						break;
                    case "漂洋过海来看你": 
						$resType = "music";
						$music_name = "漂洋过海来看你";
						$music_singer = "李行亮";
						$musicUrl   = "http://stream16.qqmusic.qq.com/35099063.mp3";
						$HQmusicUrl = "http://stream16.qqmusic.qq.com/35099063.mp3";
						break;
                    case "星空": 
						$resType = "music";
						$music_name = "星空";
						$music_singer = "理查德 克莱德曼";
						$musicUrl   = "http://wl.ibox.sjtu.edu.cn/w/27145/xingkong.mp3";
						$HQmusicUrl = "http://wl.ibox.sjtu.edu.cn/w/27145/xingkong.mp3";
						break;
               
					default:
                    	$contentStr = 
                            //		$this->xiaojo($keyword).
                            //		"【小黄鸡模式】"."\n".
                            		"我们会尽快回复"."\n".
                            		"(回复【?】查看菜单)";
						break;
                }
			}
			
			if($resType == "text"){
				$resultStr = sprintf($textTpl, $fromUsername, $toUsername, $time, $msgType, $contentStr);
			}elseif($resType == "news"){
				$resultStr = _response_news($postObj,$record);
			}elseif($resType == "multinews"){
				$resultStr = _response_multiNews($postObj,$record);
			}elseif($resType == "music"){
				$resultStr = sprintf($musicTpl, $fromUsername, $toUsername, $time, $music_name, $music_singer, $musicUrl, $HQmusicUrl);
			}
			
			echo $resultStr;
        }else{
            echo "Input something...";
        }
    }

    public function handleEvent($object)
    {
        $contentStr = "";
        switch ($object->Event)
        {
            case "subscribe":
                $contentStr = "嘿哈，欢迎来到Chloe钢琴工作室，本人一定会认真回复每一个问题和留言。"."\n"."\n".
							"回复【留言】加您的留言，我们会尽快给予回复。[得意]"."\n"."\n".
							"回复【考级】查看2014年上海音乐学院考级曲目[愉快]"."\n"."\n".
                    		"回复【1】，学琴小贴士"."[微笑]"."\n"."\n".
							"回复【2】，名家大讲堂"."[得意]"."\n"."\n".
							"回复【3】，看趣味视频"."[色]"."\n"."\n".
							"回复【4】，去活动中心"."[呲牙]"."\n"."\n".
							"回复【5】，来空中教室"."[愉快]"."\n"."\n".
							"回复【天气】，如【上海天气】，查看天气预报"."[太阳]"."\n"."\n".
							"回复【翻译】，如【翻译piano】或【翻译钢琴】，查看翻译[OK]"."\n"."\n".
							"返回菜单请回复【?】[微笑]"."\n"."\n".
                    		//"回复其他与小黄鸡聊天[呲牙]"."\n"."\n".
							"更多消息请查看名片中的历史消息";
				break;
            default :
                $contentStr = "Unknow Event: ".$object->Event;
                break;
        }
        $resultStr = $this->responseText($object, $contentStr);
        return $resultStr;
    }
    
    public function responseText($object, $content, $flag=0)
    {
        $textTpl = "<xml>
                    <ToUserName><![CDATA[%s]]></ToUserName>
                    <FromUserName><![CDATA[%s]]></FromUserName>
                    <CreateTime>%s</CreateTime>
                    <MsgType><![CDATA[text]]></MsgType>
                    <Content><![CDATA[%s]]></Content>
                    <FuncFlag>%d</FuncFlag>
                    </xml>";
        $resultStr = sprintf($textTpl, $object->FromUserName, $object->ToUserName, time(), $content, $flag);
        return $resultStr;
    }

	//小九机器人
    public function xiaojo($keyword){

        $curlPost=array("chat"=>$keyword);
        $ch = curl_init();//初始化curl
        curl_setopt($ch, CURLOPT_URL,'http://www.xiaojo.com/bot/chata.php');//抓取指定网页
        curl_setopt($ch, CURLOPT_HTTPHEADER, $header);
        curl_setopt($ch, CURLOPT_HEADER, 0);//设置header
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);//要求结果为字符串且输出到屏幕上
        curl_setopt($ch, CURLOPT_POST, 1);//post提交方式
        curl_setopt($ch, CURLOPT_POSTFIELDS, $curlPost);
        $data = curl_exec($ch);//运行curl
        curl_close($ch);
        if(!empty($data)){
            return $data;
        }else{
            $ran=rand(1,6);
            switch($ran){
                case 1:
                    return "小鸡鸡今天累了，明天再陪你聊天吧。";
                    break;
                case 2:
                    return "小鸡鸡睡觉喽~~";
                    break;
                case 3:
                    return "呼呼~~呼呼~~";
                    break;
                case 4:
                    return "你话好多啊，不跟你聊了";
                    break;
                case 5:
                    return "感谢您关注【Chloe钢琴工作室】";
                    break;
                case 6:
                    return "感谢您关注【Chloe钢琴工作室】回复【考级】可查看2014年上海音乐学院考级曲目";
                    break;
                default:
                    return "感谢您关注【Chloe钢琴工作室】";
					break;
            }
        }
    }
	
	private function weather($n){
        include("weather_cityId.php");
        $c_name=$weather_cityId[$n];
        if(!empty($c_name)){
            $json=file_get_contents("http://www.weather.com.cn/data/cityinfo/".$c_name.".html");
            return json_decode($json);
        } else {
            return null;
        }
    }
	
    private function checkSignature()
    {
        $signature = $_GET["signature"];
        $timestamp = $_GET["timestamp"];
        $nonce = $_GET["nonce"];    
                
        $token = TOKEN;
        $tmpArr = array($token, $timestamp, $nonce);
        sort($tmpArr);
        $tmpStr = implode( $tmpArr );
        $tmpStr = sha1( $tmpStr );
        
        if( $tmpStr == $signature ){
            return true;
        }else{
            return false;
        }
    }
    
    public function baiduMusic($Song, $Singer)
  {
    if (!empty($Song))
    {
      //音乐链接有两中品质，普通品质和高品质
      $music = array (
        'url' => "",
        'durl' => "");

      //采用php函数file_get_contents来读取链接内容
      $file = file_get_contents("http://box.zhangmen.baidu"
        .".com/x?op=12&count=1&title=".$Song."$$".$Singer."$$$$");

      //simplexml_load_string() 函数把 XML 字符串载入对象中
      $xml = simplexml_load_string($file, 
        'SimpleXMLElement', LIBXML_NOCDATA);

      //如果count大于0,表示找到歌曲
      if ($xml->count > 0)
      {
        //普通品质音乐
        $encode_str = $xml->url->encode;

        //使用正则表达式，进行字符串匹配，处理网址
        preg_match("/http:\/\/([\w+\.]+)(\/(\w+\/)+)/", $encode_str, $matches);

        //第一个匹配的就是我们需要的字符串
        $url_parse = $matches[0];

        $decode_str = $xml->url->decode;

        //分离字符串，截去mid
        $decode_arr = explode('&', $decode_str);

        //拼接字符串,获得普通品质音乐
        $musicUrl = $url_parse.$decode_arr[0];


        //高品质音乐
        $encode_dstr = $xml->durl->encode;
        preg_match("/http:\/\/([\w+\.]+)(\/(\w+\/)+)/", $encode_dstr, $matches_d);

        //第一个匹配的就是我们需要的字符串
        $durl_parse = $matches_d[0];

        $decode_dstr = $xml->durl->decode;
        //分离字符串，截去mid
        $decode_darr = explode('&', $decode_dstr);

        //拼接字符串,获得高品质音乐
        $musicDurl = $durl_parse.$decode_darr[0];

        //将两个链接放入数组中
        $music = array(
          'url' => $musicUrl,
          'durl' => $musicDurl
        );
        return $music;

      }

      return $music;
    }
    else
    {
      $music = "";
      return $music;
    }

  }
}

?>