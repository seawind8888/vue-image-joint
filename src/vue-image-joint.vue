<template>
  <div class="joint-ouside-container" :style="`width:${width}px`">
    <div class="image-joint-container">
      <canvas :width="width" id="jointCanvas"></canvas>
      <div class="holder-delete-container">
        <div
          class="delete-holder-item"
          v-for="(item, index) in heightList"
          :style="{height:item + 'px'}"
          :key="index"
          @click="handleDeleteJoint(index)"
        >
          <i class="delete-holder-icon" />
        </div>
      </div>
      <div v-for="(item, index) in imageCount" :key="index" class="image-joint-button-container">
        <input
          type="file"
          class="image-joint-add-file"
          accept="image/*"
          @change="handleAddImage"
          multiple
        />
        <i class="joint-middle-icon"></i>
        <p class="joint-middle-info">{{info}}</p>
      </div>
    </div>
  </div>
</template>
<script>

export default {
  name: 'VueImageJoint',
  props: {
    width: {
      type: Number,
      default: 500
    },
    info: {
      type: String,
      default: "点击添加或拖拽此处"
    },
    // 图片质量
    encoder: {
      type: Number,
      default: 0.5
    }
  },
  data() {
    return {
      imageCount: 1,
      height: 0,
      canvasY: 0,
      imageList: [],
      canvasYList: [],
      heightList: []
    };
  },
  mounted() {
    const canvas = document.querySelector("#jointCanvas");
    canvas.height = 0
  },

  methods: {
    handleAddImage(e) {
      const files = Array.from(e.target.files);
      console.log("[event]", e);
      this.filesToInstances(files, instances => {
        this.handleJoinImages(instances, finalImageUrl => {
          console.log(["finalImageUrl"], finalImageUrl);
        });
      });
    },
    filesToInstances(files, callback) {
      const length = files.length;
      let instances = [];
      let count = 0;
      files.forEach((file, index) => {
        const reader = new FileReader();
        // 把文件读为 dataUrl
        reader.readAsDataURL(file);
        reader.onload = e => {
          const image = new Image();
          image.src = e.target.result;
          //解决跨域
          // image.setAttribute('crossOrigin', 'anonymous');
          // image.src = e.target.result + '?time=' + new Date().valueOf();
          image.onload = () => {
            // 图片实例化成功后存起来
            instances[index] = image;
            count++;
            if (count === length) {
              callback(instances);
            }
          };
        };
      });
    },
    // 拼接图片
    handleJoinImages(images = []) {
      const canvas = document.querySelector("#jointCanvas");
      const context = canvas.getContext("2d");
      this.imageList = [...this.imageList, ...images];
      this.heightList = [
        ...this.heightList,
        ...images.map(item => (this.width / item.width) * item.height)
      ];
      // delete all
      if (this.heightList.length) {
        this.height = this.heightList.reduce(
          (total, current) => total + current
        );
      } else {
        this.height = 0;
      }

      canvas.height = this.height;

      if (!this.imageList.length) {
        return;
      }
      this.imageList.forEach((item, index) => {
        const height = this.heightList[index];
        let canvasY = 0;
        if (this.heightList.length >= 2 && index >= 1) {
          let canvasYList = this.heightList.slice(0, index);
          canvasY = canvasYList.reduce((a, c) => {
            return a + c;
          });
        }

        // this.canvasYList.push(this.canvasY);
        context.drawImage(item, 0, canvasY, this.width, height);
        // this.canvasY += height;
      });
    },
    getImageDataURL() {
      const canvas = document.querySelector("#jointCanvas");
      const baseUrl = canvas.toDataURL("image/jpeg", this.encoder);
      return baseUrl;
    },
    handleDeleteJoint(index) {
      this.imageList.splice(index, 1);
      this.heightList.splice(index, 1);
      this.handleJoinImages();
    }
  }
};
</script>
<style scoped>
.joint-ouside-container {
  border: 1px dashed #c0ccda;
}
.image-joint-container {
  display: flex;
  flex-direction: column;

  border-radius: 6px;
  margin: 10px;
  position: relative;
}
.image-joint-button-container {
  background-color: #fafafa;
  height: 300px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  position: relative;
}
.joint-middle-icon {
  width: 50px;
  height: 50px;
  display: block;
  background: url(data:image/png;base64,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);
  background-repeat: no-repeat;
  background-size: contain;
}
.joint-middle-info {
  color: #8a8a8a;
  font-size: 14px;

}
.image-joint-add-file {
  position: absolute;
  opacity: 0;
  cursor: pointer;
  width: 100%;
  height: 100%;
}

.delete-holder-icon {
  display: block;
  width: 50px;
  height: 50px;
  background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAYAAACtWK6eAAATwklEQVR4Xu2de+xtRXXHv0tQ8YkoKFETH1SppSriq14eBQq9cC8gVBFLgRDrxZrWmLbaVG0QgtUYiTVqFERFES2BEkVFIBSV0ooGizUNivKwtVVQK1bRiihdZl3mXPbv3HN+Z8/ea+09e+/v/EMuv5k1M9+Zz1kzs+chmAuq+lAARwN4NoCnAfgOgOsBfEBE7pqPz39TgSEroKpPBLA7gJ0AfF9Evlatj1T/oaqHAjgXwOMWVPo2AH8lIucPWRCWfdoKqOoWABsB7APgSQvUuAHAiSLyFfvbNkBU9VQAp9eQ76MAtojIz2vEZRQqUIQCyVN8AsAzaxToTgB7ishtWwFR1VMAnF0j4SzKv9swTERuzUjDqFSgFwVU9SkAvgjgkRkFOENEThVVfQKAG9MYLCM9fgDg90TEYGGgAkUqoKo7A7gOgEGSE64VkQ0GyDsBvConZSXuTwEcJiL/0jA9k1GBMAVU9eEAPpsWnHLzuUNEHmWA2KzdVqvahE0iclkbA0xLBTwVUNVdAFwF4FlN7YoFVb0FwJObGqmkIyQOItJEewUSHOY59m5jbQbIpwFsbmMopb0bwAtF5HIHWzRBBRopkOC4BsBejQzcl+hOEXm4eZC/AXBGS2PV5L8vIlc62qMpKlBLgQTH5wE8o1aC9SPdKiJ7GCA2y7ev5Q9xMGom7Gv7ZhExF8dABTpRQFV3BWBwtPUcs/JeLSIHzr6DvB7A3zrXxJaACYmzqDS3vQKquluC47cc9TlbRP6k+iX9agAHOGZgX9qPICSOitLUdgqo6qMBWN/9TWd5DhSRq6uAWEb20c/+6xVsuHWoiPyzl0HaoQIzBVT1MQmOPZ1VeYuI2Kjqvr1Y9g9VfW7K8EGOGf4fgMNF5J8cbdLUxBUIhOPPReQdM3nX7OZNkBwIwFahdnRsA4NkIz2Jo6ITNpXg+ILT97uZkj9MP+S2LWVb2A6QBMmLAFwI4H6O7UBIHMWcqilVtaMYNhrx+Lg9k/F2APuLyM3zui4EJEHyMjsk5dwQP0t7tzgncRZ2CuYSHDYh38Oxvt8DsN8iOLabg8xnqqoGyfvn5yotC2cbHA8RkS+1tMPkE1IgwWGbYm33uVdY6jmWzkEIiZf2tOOlQF9wrPQgswrSk3g1Ne3kKpDOK9mwqlPPUduDEJLcJmV8LwUSHDasWnRHQtNsVg6rqoaXTtIX5R7oSX5XROzmFAYqsFWBEuCoPcSqtlkQJD8GcDAhIR0JDlvCtaXc3jxH9hBrASTeS8A/SZD8K7vJdBVIFyzYZwDPLU+2lGvfOW7KVTZriLUAknOcPybSk+S24Ijiq6p5DvtCbnusvELWnGM+08aAJFf4RwDOIyRebTldOyXC0WgOMt+EqkpIptuvXWpeKhwugNCTuPSRyRopGQ43QAjJZPt3q4qXDocrIISkVV+ZXOIhwOEOCCGZXD9vVOGhwBECCCFp1Gcmk2hIcIQBQkgm09+zKqqqdnbcNh4W851jVQVafQdZZZxLwKsUms7fExz2hdzur/IKrT4C1ilEKCD0JHWaYPxxhgpH6BCr2uz0JOOHYFkNhwxHZ4AkT3ICgI84d5X/TY/4cKu8s7Ae5lTVLnOzOYfnxsPwYVW17uFDrAWe5MMAdvBogGSDGxwdxfQyNXTPMdOhU0CSJ3kxgAsIiVdXLM9OEBz25N+GZbePRKnQOSCEJKopy7AbCMcBImJvaXYaegGkAsnfO9/gyOFWp91nbWZjg6PTSfqidlPVowBcTEh67NVOWY8Rjt4BSZ7EILkIwAOc2srM0JM4irnKVFqtsjPk9k6HV7gDwL59DKuqFehtiFUthKoeBuASQuLVt7qzEwiHvc9hz3H0GooAJHmSwwF8xlkN8yR2zemXne3S3L1X89iLTvbsmafn+BGAg0TkqyWIXAwgCRLzJB8HsJOjOBxuOYo5MzV2zzGrZ1GAJEgOBnApIQno1U4mg+CwHzK7mqf3YVVxc5D5dlNVg+RTAB7s1KacuDsJGQhHkRcHFudBKi58PwBXEBKnnu1gZmpwmGTFApKGW4TEoWN7mAiCw96KKfpe5qIBqUByGYCHejR0ssGJe4aYgXAU/5BS8YAkSJ4P4B8JSUavdoo6ZTiKH2JV21hVCYlTp69rZupwDAqQiiexifvOdRu5RjwOtxaIFASHvXRsq1WDeZ9yEEOsOU+yD4DPEpIa6DeMEgjHRhEZ1AvHgwMkeRKD5EoAj2zYBxYloye5d/vIb6QnCDy3j5jnGBwcgxtizXmSp6d9QITE6VciwXENgN2dTJqZwcIxaECSJzFI7FKAXRwb1DzJoSJynaPN4k2l8xy28dATDqu3bTw0u4MMgxxi0ZP49rUgz3EXgM0iYvPFwYbBA1LxJPYr5T3csnHzYFZcmvRCwrG+aqMAJEHyjDQn8Rxu/SzduzVKSFT1qek1Wc+7cq05bIhqH3YHH0YDSKAnsf1CxW+JyO2JQZ7jbgCHD31YVdVyVIAQknqYBMLxQhG5vF4phhFrdIAErm6NYriV4LCPdd7DKpuQex+Z7p2iUQJCT7K4X9Fz5PM2WkAIydrOoKpPBHCt83eOXwE4cmzDqlHPQeZ/I1Q14ov7oCbuCQ77Qv74/N/QpSkMjheJyCcdbRZnatQeZKZ2ICS2YlP05jvC0Y65SQASONwqep9REBz3APiDsXuOGVaTAWRqkATC8VIR+Yd2v8vDST0pQKYCCeHwA3BygIwdElW1ibitVnlOyG1YNSnPMckhVvV3JWjibnMSW/bsZQdrgsNWq2xJ1yv8P4DjpjSsmtQy73q9JEFylfPly71s8w6E4yQR+agXbUOzM8kh1pwnsZdYvd+26BQSwhGH3eQBSXOSwUJCOOLgMMsEJOkbdJNHqCcJgkMBnDjlYRXnIEt+dAIhOcZ7v5Kq2tlxW63ynJAbHC8XkQ/G/i4Pxzo9yFxbqepeAGwlyPNkouWySUTsjuHWIcFh86antDa21sAfE461ghCQBT1MVe347uecz7jbabvWB4pU1S7xtiPA9vyZV6DnWKIkAVk+3DJI7Ffa85rTVpCoqrWXeaGNXmTYXXEcVi1Xk4Cs09NU9bnJkzzEsUOaqSNExJ6Zywqq+mYAr8tKtDqyzTk+sDraNGMQkBXtHnSrvOV6lIjYM3O1gqqeDODcWpHrRyIcK7QiIDU6k6r+DgD74u75ZuIvAOxX54lqVT0gXdi9Q43i1onCYVUdlfgdpKZK917qbJ3Urjn1DD8AsLeIfHeZ0bQz9ysAHuGY8RYReb+jvdGaogfJaFpVPSS9vuv5jrudSLR3+mxT4Jqgqg8EYHcE27Fhj0DPkakiAckULEFiE+wHZCZdL/rpInLaAkBsk+DxTvkQjgZCEpAGoqnqJgAfd4TEvId5kW3n21V1C4D3NSjeoiSEo6GQBKShcAmSSwDs2NDEfDKbjzxTRG5TVXsg6IsA7u9gm3C0EJGAtBBPVY9OnqSFlTVJbTK+GcCXATzWySi3j7QQkoC0EM+SquqLAVzU0kw1+Z0AHuZkj6tVLYUkIC0FTJAcB+BjAO7nYM7DBIdVHiryO4iTivd6khMBfLgATQmHX7PywJSjlgaJrTyd3SMkhMOzQXtsSOdqlGPOeXk2p2KEI0etmnE5B6kpVE40Vf0zAO/KSdMyLuFoKeCy5AQkSNgOISEcQW1oZglIoLiq+hoAb4vMgoedAtUlILHipiXgNwB4U0BO9BwBos6bpAfpQGRVtY2Ib3TOygB5mYh8yNkuzVUUICDB3SFdsvBeACcEZGWbHF8iIhcH2KZJDrFi+4CqPgfABQD2iM1p69vko3p+OViv2ubpQWpLlRdRVV8N4B15qVrF3r/05+Ba1a6nxATEWXhVtf1YZwGwr+pdBtvkaJB8tctMx54XAXFsYVXdFcCFAA5yNJtj6ocANojIN3MSMe5yBQiIU+9Q1aelm0fsztw+w+3ptpRb+izEWPImIA4tqap7ArDjsuZBSgj/lY7wfquEwgy5DASkZesVCMesRgbJviJi/2VoqAABaSicJSsYjlmtzIPYZRCEpGE7E5CGwqU5hz2T8KiGJpYl+xqAJwDwug/4ZgAH2GUQzuWchDkC0qCZAz3HjdaZATwPwKcbFG1ZkpvScMtuTmHIUICAZIiVhlX2nqFNyL09xw22PCwiWztxwE3uBp8Nt76fWeVJRycgGc0f6Dnsmp9DROTHs+KkD45X2P/PKOKqqFs91AzCVZH5d54Hqd0H0pzj8wAeXTtRvYg2lLINhz+fj66q9gzc9c7vEH49eRIOt2q0Dz1IDZECPYfdvXvSoourK57ELq62C6ztImuvQE9SU0kCskKoQDhOFZEz6rSTqh6VbnD0vHeLkNQQn4CsI1J6FtreBPEeVmVfB6qqp6QrhWo0a+0onLivkIqALBEoyHPYKcCTReS82l24EjHoSiF6knUag4AsEKdEOCpzEs9nEWZmCckSSAjInDCBcBwvIna6sHWgJ2ktYW0DBGTtECZiV64Nq9zgoCep3bddIhKQJOMQPMd8i9OTuDCwrhECErcrN8RzEJJ4KKo5TB6QIXoOQtIdJJMGZAxwcE4SC8tkAQmC4x67IM5rtSq36TknyVVsdfxJAhIIx7EiYs9D9xYIia/0kwNkzHBwuOULh1mbFCBTgIOQ+EIyGUCmBMccJN5vJk5qW8okAJkiHBVITgJgTyR4tvVkIPEUzde3OVmbMhyEpH0nGjUgQXD8Kh2R7XW1KrfpVZWeJFc0Z7fbIPu4JIFwHC0il8aVPM4yIcnXdpQehHAs7wiEJA+S0QFCOFZ3AEKyWqNZjFEBQjjqNzwhqafVaAAhHPUavBqLkKzWbBSAEI7VDb0sBiFZX7vBAxIExy8BHDPU1apcXAjJcsUGDUggHJtF5Mrcjjbk+IRkcesNFhDC4Y8jIdle00ECQjj84eC2lJF4EMIRBwchGbgHIRzxcFQgeSmAjzlvRxrcLuDBDLEIR3dwEJL7tB4EIEFw/ALAkVNbrcrFTFUn7UmKB0RV9wLwOQC75Tbuivj25NlVzjZHaU5VTwDwEefK2UtX9ibj95ztuporGpBAz3GYiNhzagw1FQjyJLcC2FAyJMUCQjhq9twOowVCYg+LfqfDqtTOqkhACEft9us8YhAk/5necS8OkuIAIRyd9/nsDAMhsXfcDZZiQlGAEI5i+sXKggRBYh5k35IgKQaQIDjs7fFNnJCv7O+NIgRCYnMSm8D3HooAJBAOW8r9Qu8qj7gAQZDY0q8Nt77Rt3S9A0I4+u4C7fMPguR/AOzXNyS9AkI42nfOUiwEQnKgiNzQVz17A4Rw9NXkcfkmSM4HsINjLj8CsH9fkPQCSBAcPwWwkXMOx67ZwJSqHgPgogBIDhaRf2tQpFZJOgckEA4T8LpWajCxiwJBkPwkTdw7haRTQAiHS/8bhJFASGxlsrMfws4AIRyD6NeuhQyCxIbSnY0WOgGEcLj2u0EZC4Skk/lmOCBBcNh4tFNXO6heWVhhgyDpZJdEKCDpsJOdu9jVsc16maw5ln+SpoIgsVOhoWd7wgBJcFwDYBfHHkE4HMXs2lQgJGFHp0MAIRxdd73h5BcEiV0VG3IbpjsgQXDY19RePhQNp+sNp6SBkLjfp+wKSCAcvW01GE63G1ZJgyCx9yNdn8hzA4RwDKuDllDaQEheIiIuj6y6AEI4SuhuwyxDECT3ADjWA5LWgBCOYXbMkkqdILkQwI6O5TJIThCRC9rYbAVIEBx2UKbXMwBtBGXaZgqo6mYAn3CGRAEc3waSxoAEwtH7KbJmTcxUbRUIhORkETmvSfkaAUI4mkjNNHUUCITkFSJyTp0yVONkA0I4ciVm/FwFgiCxYpySC0kWIEFwFHODRW5DMn6cAoGQvEpE3l235LUBCYTDLi8u4g6kuqIxXjcKlABJLUAIRzcdgrlsr0AgJK8VkTNXab4SEMKxSkL+PVqBPiFZFxDCEd30tF9XgUBITheR05aVYykgQXDY5cTF3Ltat3EYrwwFAiE5U0Reu6iWCwFR1Q0ALgXwCEdpvpvguMXRJk1NTAFVPQrAJQHV/jsR+Yt5u9sBoqqnAjjduQAGh21Z52qVs7BTNKeqRwL4ZEDd3yMif1q1uwYQVX0fgC3OGf93uoS4qIdRnOtIcx0rEOhJzhKRV86qsw0QVX0NgLc519M8hz2I8h/OdmmOCiDQk7xVRP7aJN4KSLqa50Znzc1j2MZD8yAMVCBEAVU9AsCnAoyfKCLnzwA5C8ArHDP5VppzFPcoo2MdaaoQBVR1U1pU8izRHQCeNAPEXmF6gZN1m4jb60D0HE6C0sxqBVT1cACfWR0zK8ZpM0BuA7B7VtLFkW0J1+Cg53AQkybyFAgYbn17Bohd47hTXnG2i22eo9gH4VvWjckHooCqHpqGW/f3KPIMEDua2CbcnOAwT8RABXpVwHO4JapqpN3dokbmOWzLup3rYKACRSjg5UnaepBvJs9BOIroFixEVQFVPQzAZS1UuWIGyPUAnpVp6OsADqLnyFSN0TtVIA23LgbwoMyM7wLw7BkgrwPw5gwDtwPYR0Q458gQjVH7UUBV904fEx+fUYLjROTCGSBG15cAPL2GgZsA2CXBvb1dXaOMjEIF1iigqrsBONduga8hzV+KyNst3vxmxWPtNjoAO88ZsVvqvg3gWhGxDY0MVGCQCqjqHwJ4PYDfXlAB+3r+attiMvvbyiO3g1SBhaYCKxRQ1ecB2Gj7BQHYSu7lAM4REXtqY1v4NXqsoHZvOp5vAAAAAElFTkSuQmCC);
  background-repeat: no-repeat;
  background-size: contain;
}
.delete-holder-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.5);
  opacity: 0;
}
.holder-delete-container {
  width: 100%;
  position: absolute;
}
.delete-holder-item:hover {
  opacity: 1;
}
/* #jointCanvas {
  height: 0;
} */
</style>
