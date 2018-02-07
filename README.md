# 快速找到卖狗地址
1. 新版 chrome 打开 https://pet-chain.baidu.com 登录自己的账号
2. F12 打开控制台
3. 复制下面代码，粘贴到 console 中, 回车
4. 拿到你已上架宠物的 petId，例如这个链接 `https://pet-chain.baidu.com/chain/detail?channel=center&petId=1855392061889709684` petId 后面的数字就是 petId
5. 控制台输入 checkPetSaleAdress('刚才拿到的 petId', 1); 记得带上引号
6. 等待搜索完毕，控制台会出现一条链接，点击之后就是狗的购买地址

-----------------------------


window.petlog = '';
function loadJquery(cb) {
  if (!window.$ ||!window.jQuery) {
    const script = document.createElement('script')
    script.src = 'https://cdn.bootcss.com/jquery/2.2.3/jquery.min.js'
    script.onload = cb;
    document.body.appendChild(script)
    return;
  }
  cb();
}

function checkPetSaleAdress(petId, page) {
  loadJquery(function() {
    _checkPetSaleAdress(petId, page);
  })
}

function _checkPetSaleAdress(petId, page) {
  const saleAdress = 'https://pet-chain.baidu.com/chain/detail?channel=market';
  page = page == null ? 1 : page;
  petId = petId + '';
  const data = {
    "pageNo": page,
    "pageSize": 10,
    "querySortType": "AMOUNT_ASC",
    "petIds": [],
    "requestId": (new Date()).getTime(),
    "appId": 1,
    "tpl": ""
  }
  console.log(`开始请求 page:${page}`);
  $.ajax({
    url: 'https://pet-chain.baidu.com/data/market/queryPetsOnSale',
    type:"POST",
    data: JSON.stringify(data),
    contentType:"application/json; charset=utf-8",
    dataType:"json",
    success: function(res){
      if (res.errorMsg === 'success' && res.data.hasData === true) {
        const petsOnSale = res.data.petsOnSale;
        let petValidCode;
        const hasPet = petsOnSale.some(function(pet) {
          petlog += pet.petId;
          if (pet.petId === petId) {
            petValidCode = pet.validCode;
            return true;
          }
          return false;
        });
        if (hasPet) {
          console.log(`${saleAdress}&petId=${petId}&validCode=${petValidCode}`);
        } else {
          checkPetSaleAdress(petId, ++page);
        }
        return;
      }
      console.error(`page: ${page} 请求错误，请重新开始`);
    },
    error: function(err) {
      console.error(`page: ${page} 请求错误，请重新开始`);
    }

  })
}
