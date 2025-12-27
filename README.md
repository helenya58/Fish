<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>致魚魚 - 完整回憶錄</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;500&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-color: #fdfaf7;
            --text-color: #4a3f35;
            --accent-color: #a68d7a;
            --card-bg: rgba(255, 255, 255, 0.96);
            --caption-bg: rgba(0, 0, 0, 0.6);
            --stage-bg: rgba(245, 240, 230, 0.6); 
        }
        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            font-family: 'Noto Sans TC', sans-serif;
            margin: 0;
            overflow-x: hidden;
        }
        .section-block {
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 80px 20px;
            opacity: 0;
            transform: translateY(30px);
            transition: all 1s ease;
        }
        .section-block.active {
            opacity: 1;
            transform: translateY(0);
        }
        .card-content {
            background: var(--card-bg);
            backdrop-filter: blur(10px);
            padding: 2rem;
            border-radius: 28px;
            max-width: 700px;
            width: 100%;
            box-shadow: 0 10px 40px rgba(74, 63, 53, 0.08);
        }
        .photo-frame {
            position: relative;
            width: 100%;
            border-radius: 16px;
            overflow: hidden;
            margin-bottom: 20px;
            background: #eee;
            aspect-ratio: 4 / 3;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            touch-action: pan-y;
        }
        .photo-frame img {
            width: 100%;
            height: 100%;
            object-fit: contain;
            pointer-events: none;
        }
        .caption-box {
            position: absolute; bottom: 0; left: 0; right: 0;
            background: var(--caption-bg);
            color: white; padding: 12px;
            font-size: 0.85rem;
            text-align: center;
            backdrop-filter: blur(4px);
            transition: opacity 0.3s ease;
        }
        .caption-hidden {
            opacity: 0;
            pointer-events: none;
        }
        .thumb-nav {
            display: flex; gap: 10px; overflow-x: auto;
            padding-bottom: 15px; scrollbar-width: none;
            margin-bottom: 15px;
        }
        .thumb-nav::-webkit-scrollbar { display: none; }
        .thumb-item {
            width: 60px; height: 60px; flex-shrink: 0;
            border-radius: 8px; overflow: hidden; opacity: 0.4;
            border: 2px solid transparent; cursor: pointer;
            transition: all 0.3s ease;
        }
        .thumb-item.active { opacity: 1; border-color: var(--accent-color); transform: scale(1.05); }
        .thumb-item img { width: 100%; height: 100%; object-fit: cover; }
        
        .story-text {
            font-size: 0.95rem;
            line-height: 1.8;
            letter-spacing: 0.03em;
            color: #3d342d;
            font-weight: 400;
        }

        .footer-stage {
            position: fixed; 
            bottom: 0; 
            left: 0; 
            width: 100%; 
            height: 65px; 
            z-index: 100; 
            pointer-events: none;
            background: var(--stage-bg);
            backdrop-filter: blur(3px);
            border-top: 1px solid rgba(255, 255, 255, 0.5);
        }
        #canvas {
            width: 100%;
            height: 100%;
            position: absolute;
            bottom: 0;
            left: 0;
            overflow: visible;
            opacity: 0.7; 
        }

        #final-overlay {
            position: fixed; inset: 0; background: black;
            z-index: 2000; opacity: 0; display: flex;
            align-items: center; justify-content: center;
            transition: opacity 4s; pointer-events: none;
        }
        #final-text {
            color: #d1c7bc; text-align: center; line-height: 2.8;
            letter-spacing: 0.12em; padding: 40px; font-weight: 300;
            font-size: 1rem;
        }
        
        /* 針對 Google Sites 的特殊顯示 */
        #sites-tip {
            position: fixed; top: 10px; right: 10px; 
            background: rgba(0,0,0,0.4); color: white;
            padding: 5px 10px; border-radius: 5px; font-size: 10px;
            z-index: 9999;
        }
    </style>
</head>
<body onclick="tryPlayMusic()">

    <!-- 修改：如果是在 Google Sites 嵌入，點擊任何地方都會啟動音樂 -->
    <div id="sites-tip">若無音樂請點擊畫面</div>

    <div id="player" class="hidden"></div>
    <div class="story-container" id="content-area"></div>
    
    <div class="footer-stage">
        <svg id="canvas" viewBox="0 0 1000 100" preserveAspectRatio="xMidYMax none"></svg>
    </div>

    <div id="final-overlay"><div id="final-text"></div></div>

    <script src="https://www.youtube.com/iframe_api"></script>
    <script>
        const storyData = [
            {
                tag: 'intro',
                text: "嗨魚魚 不知道你在打開這個網站的這刻是否還愛著我，也不知道我在分手前夕做這些是否會顯得很可笑，不過，至少在我做的這段時間，我是開心的，開心原來在以前 我們有過那麼多快樂的時光..."
            },
            {
                tag: 'part1',
                photos: [
                    {url: "https://i.postimg.cc/TKFJzg2j/1.jpg", note: "超棒的牛排 跟我超愛的酥皮濃湯！"},
                    {url: "https://i.postimg.cc/zyVk20ND/2.jpg", note: "我們去情侶聖地 體驗了一次大眾的愛情"},
                    {url: "https://i.postimg.cc/ZW2xGpYp/3.jpg", note: "魚又故意裝作好嫌棄我的樣子！幼稚！"},
                    {url: "https://i.postimg.cc/jDLh3ZTJ/4.jpg", note: "魚第一次公主抱我！真的很意外抱的起來～我超級開心！"},
                    {url: "https://i.postimg.cc/RWNTpXmf/5.jpg", note: "美顏美的好可愛～"},
                    {url: "https://i.postimg.cc/SJZrGHw5/6.jpg", note: "帶著舊安全帽的魚 好帥！ 好喜歡這個笑靨！"},
                    {url: "https://i.postimg.cc/zy6FsKDk/7.jpg", note: "魚笑得超開心 我很幸福的躺在魚的床上 享受魚的笑顏"}
                ],
                text: "雖然我記憶力很差，但你帶給我的快樂，會永遠留在我心底。"
            },
            {
                tag: 'part2',
                photos: [
                    {url: "https://i.postimg.cc/dDKcMN52/8.jpg", note: "魚拍我宜蘭的家的夜景 好開心～覺得有被重視"},
                    {url: "https://i.postimg.cc/dDmPJRf7/9.jpg", note: "魚帶我吃火鍋～雖然那時候的我還沒愛上火鍋XD"},
                    {url: "https://i.postimg.cc/V5BQz9hJ/10.jpg", note: "魚來我學校參觀～我把魚的肚子拍的好大 嘿嘿～"},
                    {url: "https://i.postimg.cc/YjfKtNPh/11.jpg", note: "沙灘車合照～真的好好玩"},
                    {url: "https://i.postimg.cc/6TLJBVPv/12.jpg", note: "超美的沙灘車合照～"},
                    {url: "https://i.postimg.cc/CdHTwC2D/13.jpg", note: "天空之鏡～但我大腿好粗..."}
                ],
                text: "這是你剛來宜蘭的時候，我很開心，也覺得很幸運，第一個人是你，你很溫柔\n後來你帶我去體驗了沙灘車，我還記得你載著我瘋狂甩尾飆車，結果被工作人員罵的情景XD，雖然被罵讓我心很慌，但有你在我身前檔著，我覺得很幸福，這是身為籠中鳥的我第一次有的驚奇體驗。"
            },
            {
                tag: 'part3',
                photos: [{url: "https://i.postimg.cc/Tpdr5mx4/14.jpg", note: "第一次開車"}],
                text: "後來你也帶著我開車，雖然我到現在還是不敢一個人上路www但我還是可以很自豪的說，我開過車！（雖然這講出去別人可能會笑死哈哈哈哈~"
            },
            {
                tag: 'part4',
                photos: [
                    {url: "https://i.postimg.cc/bZSmwpvC/15.jpg", note: "很美的房間，但我忘記是哪裡了:3"},
                    {url: "https://i.postimg.cc/KRH9bRmd/16.jpg", note: "很像職場下班後的放鬆，魚好帥"},
                    {url: "https://i.postimg.cc/k2RTgq5k/17.jpg", note: "“Sam的家” 真的很美"}
                ],
                text: "你也帶我體驗了好多，我以前一直在幻想著想要體驗看看的房間！雖然很可惜的是，還是有幾間沒去到，但至少 已經體驗過超美的幾間了！尤其是太平山那天，雖然我很想泡溫，但還好因為聖誕節，湯房都滿了，我們才能在最後體驗一起到這麼棒的房間！"
            },
            {
                tag: 'part5',
                photos: [
                    {url: "https://i.postimg.cc/SjtL23Hh/18.jpg", note: "我抱貓咪～米妮真的好可愛"},
                    {url: "https://i.postimg.cc/Z0DPB2X4/19.jpg", note: "睡得好安詳呢，我很想見見長大後的她呢"},
                    {url: "https://i.postimg.cc/F1nbd8wm/20.jpg", note: "沙灘車二次體驗～也很開心～"},
                    {url: "https://i.postimg.cc/s10PQqbC/21.jpg", note: "我很喜歡的其中一張照片，好幸福的感覺"},
                    {url: "https://i.postimg.cc/ThHjLFBw/22.jpg", note: "超好吃的生魚片！人生第一次吃到這麼好吃的"},
                    {url: "https://i.postimg.cc/23XdbMg8/23.jpg", note: "好多標本 超酷！而且這是我們僅有的一張背影合照～喜歡～"},
                    {url: "https://i.postimg.cc/jCk6wmBS/24.jpg", note: "成為畫中人！"},
                    {url: "https://i.postimg.cc/4nFv7qjm/25.jpg", note: "海邊風好大～"},
                    {url: "https://i.postimg.cc/Cd6CBX9Z/26.jpg", note: "咦～開心"},
                    {url: "https://i.postimg.cc/w3b5yZn1/27.jpg", note: "眺望遠方～"},
                    {url: "https://i.postimg.cc/mh63zJKz/28.jpg", note: "釣到不能吃的魚了！雖然我好想吃吃看這種顏色的"},
                    {url: "https://i.postimg.cc/MvM7yt8T/29.jpg", note: "現釣的山泉水魚真的超鮮美！"},
                    {url: "https://i.postimg.cc/pphfDktd/30.jpg", note: "吃到停不下來 魚不知道在幹麻"},
                    {url: "https://i.postimg.cc/nMwKjgNn/31.jpg", note: "好克難坐在高鐵地板上的我～"},
                    {url: "https://i.postimg.cc/JGDNbKWs/32.jpg", note: "魚也很克難的坐地板"}
                ],
                text: "時間不知不覺就到了2025新年，雖然是撿漏的，但我還是難掩跟你還有叔叔阿姨一同出遊的興奮，雖然我最喜歡的還是跟你單獨相處，但你願意帶我跟身邊的人多熟悉讓我很感動，不過，其實，從那之後我就知道，我們終有現在這一天的，當你叔叔問說，有沒有要結婚，你很大聲、篤定的說“沒有！“雖然你後來有解釋只是一時情急，但我知道，這是你潛意識的想法（至少我之前修心理學的課是這樣講的），所以我其實很早就沒有期待哈哈哈哈，只是偶爾還是會忍不住抱著少女心的幻想。雖說如此，那天我還是過得很開心，開啟了我對釣魚的興趣！本來很想跟你去夜釣小卷或其他魚的，不過世事難料，未來有緣再說～"
            },
            {
                tag: 'part6',
                photos: [
                    {url: "https://i.postimg.cc/YhWF491R/33.jpg", note: "我最喜歡的其中一張照片，很正式的拍了一張，我很幸福"},
                    {url: "https://i.postimg.cc/vcVn1BWS/34.jpg", note: "美美的櫻花 搭配美美的我"},
                    {url: "https://i.postimg.cc/F7LSfRjW/35.jpg", note: "美美的風景 天氣真的很好"},
                    {url: "https://i.postimg.cc/D8XGWZqB/36.jpg", note: "好颯 但紫色襯衫露出來了..."},
                    {url: "https://i.postimg.cc/N5HXyM1V/37.jpg", note: "開心 我很爽朗地將手臂靠在魚的肩膀"},
                    {url: "https://i.postimg.cc/2VZWq64P/38.jpg", note: "想親親但不敢親 嘿嘿"},
                    {url: "https://i.postimg.cc/KRMTKznC/39.jpg", note: "好耀眼的魚"},
                    {url: "https://i.postimg.cc/hXRdVBr2/40.jpg", note: "也是我愛的照片～我在相框裡～"},
                    {url: "https://i.postimg.cc/BjfK20gk/41.jpg", note: "粉色櫻花 配我粉色眼影 但魚臭臉:("}
                ],
                text: "時間回到過年出遊那幾天，我最開心的還有清境農場了，不過理由滿膚淺的，因為我覺得 在那裡拍的照片我們兩個都很好看哈哈哈～"
            },
            {
                tag: 'part7',
                photos: [
                    {url: "https://i.postimg.cc/9Ry3RyQh/42.jpg", note: "魚吃酥皮南瓜濃湯，我也吃了一口，我愛酥皮～"},
                    {url: "https://i.postimg.cc/xJLrJLTW/43.jpg", note: "我喜歡的拉花還有合照～"},
                    {url: "https://i.postimg.cc/LYjdYj88/44.jpg", note: "騎馬的魚 超級帥"}
                ],
                text: "再來是我們的生日～第一年 你帶我去吃了雅室牛排，雖然環境跟我想像中的高級餐廳不太一樣，但這是我第一次體驗除了王品以外的高級餐廳！\n中間一週年紀念時你還帶我去買了我人生中第一顆好看的包包～(不過其實這種直接幫我付錢的形式我沒很喜歡:3 我比較喜歡驚喜，雖然你怕送錯都不想自己挑來送我\n\n接下來 是你的生日，應你要求 我帶你去吃了王品，雖然我對間餐廳一開始是沒特別期待的，但實際體驗過後我改觀了，我最喜歡的是那杯拉花奶茶，雖然喝起來很苦，但那上面有我們的照片，而最後還得到了一張我們的合照～雖然我覺得拍得不好看 :3，但我喜歡他的相匡，還有氛圍，整體而言很棒！\n\n再接下來，是我的生日，也是我們可以一起過的最後一次生日了，這次是我從沒聽過的鐵板燒-阪前，這也是我第一次體驗這種高級的廚藝表演，雖然沒有合照，只有我覺得還好的生日蛋糕，而且明明該是欣賞表演的用餐方式我卻一直跟你聊天，沒好好欣賞廚師的技術（噢不過有欣賞你的帥臉），但最後還是很開心，因為我們聊了很多，而且那天很親密，我很喜歡跟你聊天 還有肢體接觸>3<"
            },
            {
                tag: 'part8',
                photos: [{url: "https://i.postimg.cc/R3HfxL1X/45.jpg", note: "我哭到眼睛紅紅的還腫起來了 魚還拍我(但被拍我其實是開心的:3"}],
                text: "最後 這是我們互相講出來自己內心深處所想的那一天，我哭得好慘哈哈哈哈，那時候雖然知道你講的有道理，畢竟我也是看很多文章的，但同時我也在想，為什麼你要這樣放棄，明明 有些東西是可以磨合調整的，是不是不夠愛才會選擇離開。\n\n但我也聽過一句話 “成年人的世界，只篩選 不教育“ 雖然我們的情況有些微不同，你說有一直暗示我，但 因為你說的，有問題會講出來，加上我小劇場很多，我選擇忽視自己的感覺，相信你有問題你真的會講，再加上我秉持著你說的“活在當下“原則，自以為只要你好我好，當雙方愛沒了才會分開，輸不知 你其實已經在為未來的人生做規劃了，而我 也早已不在你的規劃裡了。\n\n我其實一直在想，如果我沒有選擇活在當下 只體驗快樂，可能情況就會不一樣了吧，不過人生是條單行道，我們誰也不知道沒選擇的那條路未來會長什麼樣。\n\n其實我還有很多想說的，但因為這是我花了很多心思做的，如果多到你沒耐心看我會很難過，雖然我可能也不會知道:3"
            },
            {
                tag: 'part9',
                photos: [
                    {url: "https://i.postimg.cc/rDbF5CMW/46.jpg", note: "魚主動想拍我跟貓貓還有魚的狗狗跟魚合照 我很開心 很驚喜"},
                    {url: "https://i.postimg.cc/68S5CLtr/47.jpg", note: "也是跟魚魚 貓貓 狗狗合照～不過魚笑得更開心了～"},
                    {url: "https://i.postimg.cc/HrKsb4d9/48.jpg", note: "魚帶我去看下雪聖誕樹～算是圓了我半個夢～"},
                    {url: "https://i.postimg.cc/PLwfyp5W/49.jpg", note: "我想抱抱魚～"},
                    {url: "https://i.postimg.cc/nsDV2jhG/50.jpg", note: "我抱到魚了～耶～～"},
                    {url: "https://i.postimg.cc/HJyWzckR/51.jpg", note: "五寮尖登頂成功～後面有個阿杯幫我們舉牌www"},
                    {url: "https://i.postimg.cc/gwZcsX09/52.jpg", note: "只有我們兩人的五寮尖合照～天氣很好 風景很美"},
                    {url: "https://i.postimg.cc/06w5nMNR/53.jpg", note: "我的鏡頭保護貼黏在眼皮上，真的很愛有黏性的東西"},
                    {url: "https://i.postimg.cc/cgGxbxvj/54.jpg", note: "騎鐵道自行車～踩得好累 魚還覺得是我不運動的關係"},
                    {url: "https://i.postimg.cc/r02VPVd6/55.jpg", note: "抱抱～我愛的貼貼"},
                    {url: "https://i.postimg.cc/hzRD5DJg/56.jpg", note: "我好性感～魚好色"},
                    {url: "https://i.postimg.cc/wyYqGqtj/57.jpg", note: "魚愛的多拉a夢"},
                    {url: "https://i.postimg.cc/sQPjFkSg/58.jpg", note: "漁人碼頭煙火秀"},
                    {url: "https://i.postimg.cc/mzWL5LcM/59.jpg", note: "101下的盛世美顏"},
                    {url: "https://i.postimg.cc/ft7z4GXV/60.jpg", note: "雙人電輔自行車～好玩 魚好厲害 很會騎"},
                    {url: "https://i.postimg.cc/d7yQ6k0Y/61.jpg", note: "超美咖啡廳～"},
                    {url: "https://i.postimg.cc/nX5cqYpQ/62.jpg", note: "跟結痂吃飯～以前比較喜歡兩個人，現在有點難過沒多約一點"}
                ],
                text: "總之 我就以我喜歡的、對我來說很有意義的照片來做結尾吧！"
            },
            {
                tag: 'fade',
                text: "最後，我想跟你說，你曾是我的全世界，謝謝你出現在我的生命中，教會了我什麼是愛，主動想為一個人花錢是什麼樣的感覺，也讓情緒穩定，或者說長期壓抑的我感受了一次歇斯底里，控制不住自己是什麼體驗。"
            },
            {
                tag: 'part10',
                photos: [
                    {url: "https://i.postimg.cc/JycsW6f8/63.jpg", note: "山中仙境～"},
                    {url: "https://i.postimg.cc/HJ4rCZqg/64.jpg", note: "可惜沒有去..."},
                    {url: "https://i.postimg.cc/4KbYTLDT/65.jpg", note: "有一點點點的太陽～"},
                    {url: "https://i.postimg.cc/2LjvJRyd/66.jpg", note: "一開始停車等日出的地方～"},
                    {url: "https://i.postimg.cc/mzyC82Lp/67.jpg", note: "魚開車辛苦了～戳魚鼻孔 嘿嘿"},
                    {url: "https://i.postimg.cc/FkFcq5zj/68.jpg", note: "魚開車辛苦了～我陪你一起睡"},
                    {url: "https://i.postimg.cc/G8v2L80B/69.jpg", note: "我愛親親 但魚好討厭 一直擺出這個表情"},
                    {url: "https://i.postimg.cc/2bn6kbp8/70.jpg", note: "雨中漫步～好浪漫"},
                    {url: "https://i.postimg.cc/Y450BNP9/71.jpg", note: "我在人間仙境～"},
                    {url: "https://i.postimg.cc/Ny3Fv8ng/72.jpg", note: "我們好厲害 打到很多彩票～"},
                    {url: "https://i.postimg.cc/jWp2Y61q/73.jpg", note: "最後一次一起租車的合照"}
                ],
                text: "最後，謝謝你最後給我一星期的溫柔，讓我可以整理好自己的心情，做出這樣的東西。\n太平山之旅雖然有遺憾，但這份不完美，可能就是留著給下一個人圓滿的吧。"
            }
        ];

        function renderStory() {
            const container = document.getElementById('content-area');
            storyData.forEach((sec, i) => {
                const div = document.createElement('div');
                div.className = 'section-block';
                div.id = `sec-${i}`;
                const card = document.createElement('div');
                card.className = 'card-content';
                
                if (sec.photos) {
                    const frame = document.createElement('div');
                    frame.className = 'photo-frame';
                    const img = document.createElement('img');
                    img.src = sec.photos[0].url;
                    img.id = `img-${i}`;
                    img.loading = "lazy";
                    const cap = document.createElement('div');
                    cap.className = 'caption-box';
                    cap.id = `cap-${i}`;
                    cap.textContent = sec.photos[0].note;
                    frame.onclick = () => cap.classList.toggle('caption-hidden');
                    let touchStartX = 0;
                    frame.addEventListener('touchstart', e => { touchStartX = e.changedTouches[0].screenX; }, {passive: true});
                    frame.addEventListener('touchend', e => {
                        const touchEndX = e.changedTouches[0].screenX;
                        const diff = touchStartX - touchEndX;
                        if (Math.abs(diff) > 50) {
                            const currentIdx = sec.photos.findIndex(p => p.url === img.src);
                            let nextIdx = diff > 0 ? currentIdx + 1 : currentIdx - 1;
                            if (nextIdx >= 0 && nextIdx < sec.photos.length) updatePhoto(i, nextIdx);
                        }
                    }, {passive: true});
                    frame.appendChild(img);
                    frame.appendChild(cap);
                    card.appendChild(frame);
                    if (sec.photos.length > 1) {
                        const nav = document.createElement('div');
                        nav.className = 'thumb-nav';
                        nav.id = `nav-${i}`;
                        sec.photos.forEach((p, pi) => {
                            const t = document.createElement('div');
                            t.className = `thumb-item ${pi===0?'active':''}`;
                            t.onclick = (e) => { e.stopPropagation(); updatePhoto(i, pi); };
                            t.innerHTML = `<img src="${p.url}" loading="lazy">`;
                            nav.appendChild(t);
                        });
                        card.appendChild(nav);
                    }
                }
                const text = document.createElement('div');
                text.className = 'story-text whitespace-pre-wrap';
                text.textContent = sec.text;
                card.appendChild(text);
                div.appendChild(card);
                container.appendChild(div);
            });
            const spacer = document.createElement('div'); spacer.style.height = '80vh';
            container.appendChild(spacer);
            document.getElementById('final-text').innerHTML = "最後的最後，真的最後，雖然我現在還是很痛，但<br>我希望你可以找到你的真愛，找到一個你認定要跟她過後半餘生的人，幸福快樂。";
        }

        function updatePhoto(sectionIdx, photoIdx) {
            const sec = storyData[sectionIdx];
            const p = sec.photos[photoIdx];
            const img = document.getElementById(`img-${sectionIdx}`);
            if(img) img.src = p.url;
            const cap = document.getElementById(`cap-${sectionIdx}`);
            if(cap) cap.textContent = p.note;
            const nav = document.getElementById(`nav-${sectionIdx}`);
            if (nav) {
                [...nav.children].forEach((c, idx) => {
                    c.classList.toggle('active', idx === photoIdx);
                });
            }
        }

        let player;
        let isPlaying = false;
        function onYouTubeIframeAPIReady() {
            player = new YT.Player('player', {
                height: '0', width: '0', videoId: 'p-fVzD-9f8o',
                playerVars: { 'autoplay': 1, 'controls': 0, 'loop': 1, 'playlist': 'p-fVzD-9f8o' },
                events: {
                    'onReady': (event) => {
                        // 嘗試自動播放
                        event.target.playVideo();
                    }
                }
            });
        }

        function tryPlayMusic() {
            if (player && player.playVideo && !isPlaying) {
                player.playVideo();
                isPlaying = true;
                const tip = document.getElementById('sites-tip');
                if(tip) tip.style.display = 'none';
            }
        }

        function loop() {
            const scroll = window.pageYOffset;
            const max = do
