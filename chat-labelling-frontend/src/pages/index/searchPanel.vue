<template>
  <div style="width:100%;height:100%">
    <div v-show="showFrame" style="width:100%;height:100%">
      <Button type="primary" @click="showFrame=false">
        <Icon type="ios-arrow-back" />Back to results
      </Button>
      <div style="width:100%;height: calc( 100% - 50px );margin-top:10px">
        <Spin size="large" v-if="frameLoading" fix>
          <Icon type="ios-loading" size="50" class="spin-icon-load"></Icon>
          <div>Loading page...</div>
        </Spin>
        <iframe
          name="frame"
          :src="pageLink"
          style="width:100%;height:100%"
          ref="frame"
          frameborder="0"
        />
      </div>
    </div>
    <div v-show="!showFrame">
      <Spin size="large" v-if="loading" fix>
        <div :style="{ fontSize: fontSize }">Please search first</div>
      </Spin>
      <Spin size="large" v-if="searching">
        <Icon type="ios-loading" size="200" class="spin-icon-load"></Icon>
        <div>正在搜索...</div>
      </Spin>
      <CheckboxGroup v-model="selected" @on-change="onCheckGroupChange">
        <Collapse simple v-model="collapse">
          <Panel v-for="(actions,name) in data" v-if="name==='Answer'" :key="name">
            <strong>产品搜索结果</strong>
            <div slot="content">
              <List item-layout="vertical" size="small">
                <ListItem v-for="(item,index) in actions" :key="name+'-'+index">
                  <Checkbox :label="name+'-'+index" :disabled="(disable ===true && item.selected === 'false') || (active.indexOf(name) < 0)" :value="item.selected" @input="toggleSelected(item)">
                  <img :src=item.image alt="" height="50px" width="50px">
                  <span style="color:black">{{item.title}}</span>
                  </Checkbox>
                  <a v-if="item.link" target="_blank" :href="item.link">打开产品页面</a>
                  <Divider type="vertical" />
                  <a v-if="item.link" @click="copyLink(item.link)">复制链接</a>
                </ListItem>
              </List>
<!--              <div-->
<!--                v-else-->
<!--                v-for="(item,index) in actions"-->
<!--                :key="name+'-'+index"-->
<!--                style="padding:10px;width:50%;float:left"-->
<!--              >-->
<!--                <Checkbox :label="name+'-'+index" :disabled="active.indexOf(name)<0">{{item.title}}</Checkbox>-->
<!--              </div>-->
<!--              <div class="clearfix"></div>-->
            </div>
          </Panel>
          <Panel v-for="(actions,name) in data" v-if="name==='Aspects'" :key="name">
            <strong>明晰问题提问词</strong>
            <div slot="content">
              <List item-layout="vertical" size="small">
                <ListItem v-for="action in actions" :key="action">
                  <Checkbox :label="action">
                    <span style="color:black">{{action}}</span>
                  </Checkbox>
                </ListItem>
              </List>
            </div>
          </Panel>
        </Collapse>
      </CheckboxGroup>
    </div>
  </div>
</template>
<script>

const arrayEquals = (arr1, arr2) => {
  // 比较两个数组，内容相同，顺序也相同，返回true，内容相同顺序不同返回‘orderChange’
  if (arr1.length !== arr2.length) return false
  if (JSON.stringify(arr1) === JSON.stringify(arr2)) return true
  let difference = arr1.concat(arr2).filter(v => !arr1.includes(v) || !arr2.includes(v))
  return difference.length === 0 ? 'orderChange' : false
}
export default {
  props: ['value', 'activeActions', 'loading', 'data', 'searchResultConfig', 'searching'],
  model: {
    prop: 'value',
    event: 'change'
  },
  created () {
    if (this.value.length > 0) {
      this.value.map(val => {
        this.selected.push(val.itemId)
      })
    }
  },
  mounted () {
    this.bindLoadEvent()
  },
  watch: {
    selected (newVal, oldVal) {
      if (newVal.length >= this.maxSelection) {
        this.disable = true
      } else {
        this.disable = false
      }
    },
    value (newVal, oldVal) {
      let equals = arrayEquals(newVal.map(v => v.itemId), oldVal.map(v => v.itemId))
      if (equals === true) return
      this.selected = newVal.map(v => v.itemId)
      // value改动，说明model的值改动,就可以触发事件
      // 事件有两种，一种是改了顺序，一种是改了选项
      if (equals === false) { this.$emit('on-change', newVal) }
      if (equals === 'orderChange') { this.$emit('on-order-change', newVal) }
    },
    activeActions (newVal) {
      if (!newVal) return
      let currentActive = []
      for (let activeItem in this.searchResultConfig) {
        this.searchResultConfig[activeItem].indexOf(newVal[0]) >= 0 && currentActive.push(activeItem)
      }
      this.active = [...currentActive]
    },
    data (newVal) {
      this.collapse = Object.keys(newVal)
    }
  },
  data () {
    return {
      selected: [],
      collapse: [],
      active: '',
      pageLink: '',
      showFrame: false,
      frameLoading: false,
      fontSize: '44px',
      maxSelection: 7,
      disable: false
    }
  },
  methods: {
    toggleSelected (item) {
      console.log('item:')
      console.log(item.id)
      for (let i = 0; i < this.data['Answer'].length; i++) {
        console.log(this.data['Answer'][i])
        if (item.id === this.data['Answer'][i].id) {
          this.data['Answer'][i].selected = this.data['Answer'][i].selected === 'false' ? 'true' : 'false'
        }
      }
      console.log(this.data)
    },
    showWebPage (link) {
      this.frameLoading = true
      this.showFrame = true
      this.$nextTick(() => {
        this.pageLink = link
      })
      // 一分钟后，不管什么情况，停止loading状态
      setTimeout(() => {
        this.frameLoading = false
      }, 1000 * 30)
    },
    copyLink (link) {
      this.$copyText(link).then((e) => {
        this.$Message.info('Copied!')
      }, (e) => {
        this.$Message.info('Can not copy!')
      })
    },
    bindLoadEvent () {
      const iframe = this.$refs.frame
      iframe.onload = () => {
        this.frameLoading = false
      }
      iframe.onreadystatechange = () => {
        if (iframe.readyState === 'interactive' || iframe.readyState === 'complete' || iframe.readyState === 'loaded') {
          this.frameLoading = false
        }
      }
    },
    getActive () {
      return this.active
    },
    onCheckGroupChange (checked) {
      let checkedData = []
      console.log(this.data['Answer'])
      checked.map(value => {
        let arr = value.split('-')
        if (arr[0] === 'Answer') {
          console.log(this.data[arr[0]][arr[1]])
          checkedData.push({...this.data[arr[0]][arr[1]], itemId: value})
        } else {
          checkedData.push({title: value, itemId: value, id: value})
          console.log('nie')
          console.log({title: value, itemId: value})
        }
      })
      console.log('here!')
      console.log(checkedData)
      this.$emit('change', checkedData)
      this.$emit('changeBackupForParent', checkedData)
    },
    reset () {
      this.selected = []
      this.collapse = []
      this.active = ''
      this.showFrame = false
      this.pageLink = ''
      this.frameLoading = false
    }
  }

}
</script>
