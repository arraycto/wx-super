<template>
  <div class="posre borderbox wrapper h100p" :style="{height: mainHeight+'px'}">
    <NavBarByUser
      @cancelLoginGuide="cancelLoginGuide"
      :isLogin="isLogin"
      :isShowLoginGuide="isShowLoginGuide"
      :isShowCardCase="true"
      @loginSuccess="loginSuccess"
      @loginFailed="loginFailed"
      :avatarUrl.sync="avatarUrl"
    />
    <scroll-view
      :style="{height: scrollContentHeight+'px'}"
      class="dynamic-content"
      :scroll-y="true"
      :enable-back-to-top="true"
      :scroll-anchoring="true"
      @scrolltoupper="scrolltoupper"
      @scrolltolower="scrolltolower"
    >
      <div class="w100p" style="position: relative; z-index: 2">
        <SelfSwiper
          :imgUrls="banners"
          :self_class="'h211'"
          :isShowMask="true"
          @swipclick="preview"
          :isShowDots="false"
        />
      </div>

      <div class="bgfff">
        <div class="fs18 cfff disflex pl15 pr16 company-title">
          <img
            :src="company_msg.company_logo || 'https://hq-one-stand.oss-cn-shenzhen.aliyuncs.com/yimai_photos/user/card1_user.png'"
            mode="aspectFill"
            alt
            class="w60 h60 bradius5 mr10 bgfff"
          />
          <div class="flex1">
            <div class="disflex jsbet align-cen">
              <b class="pb10 fbold over_1 w190">{{company_msg.company_name}}</b>
              <AuthenticationTag />
            </div>
            <div class="disflex jsbet mt8">
              <div class></div>

              <div class="disflex align-cen">
                <button
                  class="w24 pl0 pr0 bgfff contentbox textr disflex align-cen mr23"
                  open-type="share"
                  hover-class="other-button-hover"
                >
                  <img
                    src="https://hq-one-stand.oss-cn-shenzhen.aliyuncs.com/yimai_photos/user/share2.png"
                    alt
                    class="w20 h20 mr7"
                    style="flex: 0 0 40upx"
                  />
                </button>
                <img
                  src="https://hq-one-stand.oss-cn-shenzhen.aliyuncs.com/product-index/20190418161144.png"
                  alt
                  class="w20 h20"
                  @click="collect"
                  v-if="isCollect"
                />
                <img
                  src="https://hq-one-stand.oss-cn-shenzhen.aliyuncs.com/yimai_photos/user/star.png"
                  alt
                  class="w20 h20 posre"
                  @click="collect"
                  v-if="!isCollect"
                />
              </div>
            </div>
          </div>
        </div>
      </div>

      <div>
        <div class="nav_top">
          <div class="posre pl15 pr42 bgfff">
            <scroll-view scroll-x class="nav" scroll-with-animation>
              <div class>
                <div
                  class="nav-item"
                  v-for="(navItem,idx) in menu"
                  :key="idx"
                  :class="menu_id == navItem.productsTypeId ?'active':''"
                  @click="menu_tap(navItem.productsTypeId)"
                >{{navItem.productsTypeName}}</div>
              </div>
            </scroll-view>
            <img
              src="https://hq-one-stand.oss-cn-shenzhen.aliyuncs.com/yimai_photos/user/search.png"
              alt
              @click="toSearchGoods"
              class="w20 h20 menu_more posab"
            />
          </div>
        </div>

        <div class="bgf5f6 pl15 pr15 pb23" v-if="prod_lists.length">
          <getPhoneNumberGoods
            v-for="(v,k) in prod_lists"
            :key="k"
            goodsType="product"
            @next="toProdDetail(v.productsId)"
            @loginGuide="loginGuide"
            :goodInfo="v"
          ></getPhoneNumberGoods>
        </div>
        <div v-else>
          <NoData>暂无商品</NoData>
        </div>
        <!--lists end-->
      </div>

      <!--右边悬浮框-->
      <!--    <div class="posfix right11 add_white bottom10 w40 h40 bg_line_blue bradius50p textc lh40" @click="right_float_show = !right_float_show">-->

      <!--    </div>-->

      <!--bottom-->
      <div class="textc lh42 fs12 ca8 bgf5f6" v-if="nodata && prod_lists.length">- 汉全科技集团出品 -</div>

      <!-- <div class="index_float_right trans2 overhidden">
        <RightFloat :isShow="isShow"  @clickRightRowEvent="clickRightRowEvent"></RightFloat>
        <div :class="{fadeInRight: !isShow, fadeOutRight: isShow}" style="width: 40rpx;right: 0;border-top-left-radius: 10rpx;border-bottom-left-radius: 10rpx"  class="animated posfix fs10 ca8 textc floats trans2 bottom30  shadow_gray bgfff pb15"> 
          <div class="pt16" @click="clickRightRowEvent">
            <img src="https://hq-one-stand.oss-cn-shenzhen.aliyuncs.com/yimai_photos/user/right_row.png" alt="" style="width: 30rpx;transform: rotate(180deg);height: 30rpx;">
          </div>
        </div>
      </div>-->
    </scroll-view>
    <FloatButtons
      class="index_float_right trans2 overhidden"
      :isShowTalk="isShowTalk"
      :avatarUrl="targetAvatarUrl"
      @talk="talk"
      @loginGuide="loginGuide"
    ></FloatButtons>
  </div>
</template>

<script>
import RightFloat from "@/components/rightFloat"; // 订单项
import getPhoneNumberGoods from "@/components/getPhoneNumberGoods"; //
import WXAJAX from "../../utils/request";
import NoData from "@/components/noData";
import utils from "@/utils";
import SelfSwiper from "@/components/swiper"; //
import AuthenticationTag from "@/components/AuthenticationTag"; //
import FloatButtons from "@/components/FloatButtons.vue";
import NavBarByUser from "@/components/NavBarByUser.vue";
import { mapState } from "vuex";
import HandleLogin from "@/utils/handleLogin";
import { addShareRecord } from "@/utils/behavior";

export default {
  name: "",
  components: {
    RightFloat,
    NoData,
    getPhoneNumberGoods,
    SelfSwiper,
    AuthenticationTag,
    FloatButtons,
    NavBarByUser
  },
  data() {
    return {
      canClickStar: true, // 是否可以点击star按钮
      menu: [{ productsTypeName: "全部", productsTypeId: "-10" }],
      menu_id: "-10",
      prod_lists: [],
      nodata: false, //是否已经没有数据
      COMPANYID: 0,
      company_msg: {
        admin_logo: "",
        company_logo: "",
        company_name: ""
      },
      currentPage: 1,
      isCollect: false, //是否收藏
      collectionId: 0,
      page: 1,
      isLoading: false, //是否在加载
      right_float_show: false, //右边悬浮框
      banners: [
        "https://hq-one-stand.oss-cn-shenzhen.aliyuncs.com//product-index/20190418160834.png"
      ],
      isShow: true,

      avatarUrl: "", //用户的头像地址,
      isLogin: HandleLogin.returnIsLogin() || false, //是否已经登录
      isShowTalk: true, //是否显示右边侧边栏的聊一聊
      isShowLoginGuide: true, //是否显示登录引导页,
      targetAvatarUrl: "", //当前目标的人物头像
      scrollContentHeight: 0, //中间滚动区域的高度
      mainHeight: 0 //整体高度
    };
  },
  onShow() {
    //获取当前的公司
    let lastCompanyid = this.COMPANYID;
    this.COMPANYID = wx.getStorageSync("COMPANYID") || 0;
    //this.getProds();

    if (lastCompanyid != this.COMPANYID || !this.COMPANYID) {
      this.prod_lists = [];
      this.page = 1;
      this.getcompany();
      this.getTypeMenus();
      this.menu_tap("-10");
      this.getAd();
    } else {
      this.prod_lists = [];
      this.page = 1;
      this.getcompany();
      this.getTypeMenus();
      this.getProds();
    }
    //this.getMeuns();
    this.isLogin = HandleLogin.returnIsLogin() || false;
    this.avatarUrl = wx.getStorageSync("avatarUrl") || "";
  },
  computed: {
    ...mapState({
      currentCompany: state => state.currentCompany
    })
  },
  watch: {
    currentCompany: {
      deep: true,
      immediate: true,
      handler(newVal) {
        if (newVal) {
          if (newVal.userId === wx.getStorageSync("userId")) {
            this.isShowTalk = false;
          } else {
            this.isShowTalk = true;
            this.targetAvatarUrl = newVal.logo;
          }
        }
      }
    }
  },
  onShareAppMessage(e) {
    let COMPANYID = wx.getStorageSync("COMPANYID") || 0;
    let CARDID = wx.getStorageSync("CARDID") || 0;
    let uuid = CARDID + "" + new Date().getTime();
    addShareRecord(this.COMPANYID, 3, "", uuid).then(
      res => {},
      err => {}
    );
    this.addGoodsForwardRecord(CARDID);
    return {
      path: `/pages/appointment/main?cardId=${CARDID}&companyId=${COMPANYID}&goType=1&shareId=${uuid}`,
      success(e) {
        wx.showShareMenu({
          withShareTicket: true
        });
      },
      fail(e) {
        console.log("失败分享名片--- ", e);
      }
    };
  },
  onPageScroll(e) {
    if (e.scrollTop > 200) {
      wx.setNavigationBarColor({
        frontColor: "#000000",
        backgroundColor: "#ffffff",
        animation: {
          duration: 200,
          timingFunc: "easeIn"
        }
      });
    } else {
      wx.setNavigationBarColor({
        frontColor: "#ffffff",
        backgroundColor: "#ffffff",
        animation: {
          duration: 200,
          timingFunc: "easeIn"
        }
      });
    }
  },
  async onPullDownRefresh() {
    // to doing..
    // 停止下拉刷新
    wx.showNavigationBarLoading();
    this.page = 1;
    this.prod_lists = [];
    this.nodata = false;
    this.isLoading = false;
    this.getcompany();
    this.getTypeMenus();
    this.getAd();
    this.menu_tap(this.menu_id || "-10");
    // wx.showLoading();
    wx.stopPullDownRefresh();
    setTimeout(function() {
      // wx.hideLoading();
      wx.hideNavigationBarLoading();
    }, 300);
  },
  async mounted() {
    let a = await utils.systemIfo();
    let navHeight = getApp().globalData.navHeight;
    this.mainHeight = a.windowHeight;
    this.scrollContentHeight = a.windowHeight - navHeight;
  },
  methods: {
    //下拉刷新
    scrolltoupper(e) {
      this.page = 1;
      this.prod_lists = [];
      this.nodata = false;
      this.isLoading = false;
      this.getcompany();
      this.getTypeMenus();
      this.getAd();
      this.menu_tap(this.menu_id || "-10");
      // wx.showLoading();
    },
    //上拉加载更多
    scrolltolower(e) {
      // this.showCompany = false;
      // wx.showLoading({
      //   title : '加载中...',
      // });
      let v = this;
      v.getProds();
    },
    //隐藏登录引导
    cancelLoginGuide() {
      this.isShowLoginGuide = false;
    },
    //需要登录引导的回调
    loginGuide() {
      this.isShowLoginGuide = true;
    },
    //聊一聊
    talk() {
      wx.navigateTo({
        url:
          "../IM/main?userId=" +
          this.currentCompany.userId +
          "&logo=" +
          this.currentCompany.logo +
          "&type=2&cardId=" +
          this.currentCompany.cardId +
          "&name=" +
          this.currentCompany.name +
          "&wxCode=" +
          (this.currentCompany.personalWx || "") +
          "&phone=" +
          this.currentCompany.phone
      });
    },
    //头像登录成功的回调
    loginSuccess(url) {
      this.isLogin = true;
      this.hasCard = wx.getStorageSync("hasCard") || false;
      this.avatarUrl = wx.getStorageSync("avatarUrl") || "";
    },
    loginFailed() {
      this.isShowLoginGuide = true;
    },
    toSearchGoods() {
      wx.navigateTo({
        url:
          "/pages/appointmentPack/searchGoods/main?companyId=" + this.COMPANYID
      });
    },
    clickRightRowEvent() {
      this.isShow = !this.isShow;
    },
    // 获取预约分类
    getTypeMenus() {
      WXAJAX.POST(
        {
          companyId: this.COMPANYID
        },
        "",
        "/products/selectProductsTypeList/V2"
      ).then(res => {
        if (res) {
          this.menu = [
            { productsTypeName: "全部", productsTypeId: -10 },
            ...res
          ];
        } else {
          this.menu = [{ productsTypeName: "全部", productsTypeId: -10 }];
        }
      });
    },
    preview(idx) {
      this.previewImages(this.banners, this.banners[idx]);
    },
    getAd() {
      this.getPlateAds(2)
        .then(res => {
          this.banners = res
            ? res.map(val => val.photo)
            : [
                "https://hq-one-stand.oss-cn-shenzhen.aliyuncs.com//product-index/20190418160834.png"
              ];
        })
        .catch(() => {
          this.banners = [
            "https://hq-one-stand.oss-cn-shenzhen.aliyuncs.com//product-index/20190418160834.png"
          ];
        });
    },
    getcompany() {
      //获取公司信息
      let v = this;
      WXAJAX.POST(
        {
          companyId: v.COMPANYID
        },
        "",
        "/goods/getCompanyInfo/V2"
      )
        .then((data, code) => {
          if (data) {
            v.company_msg = {
              admin_logo: data.userLogo || "",
              company_logo: data.companyLogo || "",
              company_name: data.companyName || ""
            };
            v.company_msg = {
              company_logo: data.companyLogo,
              company_name: data.companyName
            };
            v.isCollect = data.isCollection == 1;
          }
        })
        .catch(err => {
          this.company_msg = {};
        });
    },
    checkCollect() {
      //
      WXAJAX.checkCollect({
        itemType: 3,
        itemId: this.COMPANYID
      })
        .then(data => {
          this.isCollect = data.status;
          this.collectionId = data.collectionId || 0;
        })
        .catch(err => {});
    },
    getProds() {
      //获取产品
      let v = this;
      if (v.isLoading) {
        wx.hideLoading();
        return;
      }
      v.isLoading = true;
      WXAJAX.POST(
        {
          productsTypeId: v.menu_id == -10 ? "" : v.menu_id,
          pageNum: v.page,
          companyId: wx.getStorageSync("COMPANYID"),
          cardId: wx.getStorageSync("CARDID")
        },
        "",
        "/products/selectProductsPage/V2"
      )
        .then(data => {
          wx.hideLoading();
          if (data) {
            // wx.showToast({
            //     title: '更新成功',
            //     icon: 'success',
            //     duration: 1000
            //   })
            data.pageInfo.list.forEach(function(i, k) {
              if (i.productsPhoto) {
                i.prodLogo = i.productsPhoto.split(",")[0];
              } else {
                i.prodLogo = "";
              }
              if (i.price) {
                let price;

                try {
                  price = JSON.parse(i.price);
                } catch (e) {
                  price = parseInt(i.price) || 0;
                }

                if (
                  Object.prototype.toString.call(price).toLowerCase() ===
                  "[object number]"
                ) {
                  i.price = price.toFixed(2);
                } else {
                  i.price = price.map(val => parseInt(val)).join("~");
                }
              } else {
                i.price = "";
              }
            });

            v.prod_lists = [...v.prod_lists, ...data.pageInfo.list];
            v.page++;
            setTimeout(function() {
              v.isLoading = false;
            }, 500);
          } else {
            // // v.prod_lists = [] ;
            // setTimeout(function () {
            //   v.nodata = true ;
            //   v.isLoading = false ;
            // },500);
            // wx.showToast({
            //     title: '没有更多信息',
            //     icon: 'success',
            //     duration: 1000
            // })
          }
        })
        .catch(err => {
          console.log(err);
          v.prod_lists = [];
          wx.hideLoading();
          if (err.code == 204) {
            v.nodata = true;
          }
          setTimeout(function() {
            v.isLoading = false;
          }, 500);
        });
    },
    menu_tap(id) {
      this.menu_id = id || "";
      this.page = 1;
      this.nodata = false;
      this.isLoading = false;
      this.prod_lists = [];
      // wx.showLoading() ;
      this.getProds();
    },
    toProdDetail: utils.throttle(function(id) {
      wx.navigateTo({
        url: "/pages/appointmentPack/productDetail/main?goodId=" + id
      });
    }, 1000),
    collect() {
      //收藏企业
      // 没有登录，提醒登录
      if (!HandleLogin.returnIsLogin()) {
        this.loginGuide();
        return;
      }

      if (this.canClickStar) {
        this.canClickStar = false;
        let _url = "",
          _request = {};
        if (this.isCollect) {
          //取消收藏
          _url = "/personal/delCollection";
          _request = {
            itemType: 3,
            itemId: this.COMPANYID
          };
        } else {
          //添加收藏
          _url = "/personal/addCollection";
          _request = {
            itemType: 3,
            itemId: this.COMPANYID
          };
        }
        WXAJAX.changeCollect(_request, _url)
          .then(data => {
            console.log(data);
            if (data) {
              this.isCollect = !this.isCollect;
              wx.showToast({
                // 这玩意要不要，跟产品协商一下，感觉体验不好
                title: this.isCollect ? "收藏成功！" : "取消收藏！",
                icon: "none",
                duration: 2000
              });
            }
            setTimeout(() => {
              // 防止用户多次点击，具体时间咨询产品
              this.canClickStar = true;
            }, 200);
          })
          .catch(err => {
            setTimeout(() => {
              this.canClickStar = true;
            }, 200);
          });
      }
    }
  }
  // onReachBottom() {
  //   this.showCompany = false;
  //   wx.showLoading({
  //     title : '加载中...',
  //   });
  //   let v = this ;
  //   v.getProds();
  // },
};
</script>

<style>
.company-title {
  margin-top: -60upx;
  position: relative;
  z-index: 3;
}

.menu_more {
  right: 32upx;
  top: 0;
  bottom: 0;
  margin: auto;
}

.nav {
  height: 90upx;
  width: 100%;
  line-height: 88upx;
  font-size: 28upx;
  white-space: nowrap;
  /* position: sticky;
    top: 0; */
  /*   position: fixed;
       top: 0;
       left: 0;
       z-index: 99;*/
}
.nav_top {
  position: sticky;
  background: white;
  /*padding-top: 140upx;*/
  /*margin-top: -140upx;*/
  top: 0;
}
.nav-item {
  /*width: 20%;*/
  display: inline-block;
  text-align: center;
  margin-right: 48upx;
}

.nav-item.active {
  border-bottom: 4upx solid #00a0e9;
  color: #00a0e9;
  font-weight: bold;
}

.prod_float_right > div {
  bottom: 120upx;
}
</style>
