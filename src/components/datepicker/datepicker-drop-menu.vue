<script>
import Vue from 'vue'
import PositionHelper from '../../mixins/positionHelper'
import Emitter from '../../mixins/emitter'

let DropMenu

export default {
  mixins: [PositionHelper, Emitter],

  props: {
    select: {},
    isFocus: Boolean
  },

  data () {
    return {
      isTransitioning: false
    }
  },

  updated () {
    DropMenu.$forceUpdate()
  },

  watch: {
    selectIndex () {
      DropMenu.$forceUpdate()
    }
  },

  render (h) {
    return h()
  },

  mounted () {
    const _this = this
    const parent = this.select

    DropMenu = new Vue({
      data: {
        parent: parent
      },
      render (h) {
        return h('div', {
          class: 't-datepicker__wrapper'
        }, [h('div', {
          class: [
            't-datepicker__arrow-up',
            _this.arrowRight ? 't-datepicker__arrow-up--right' : '',
            _this.arrowTop ? 't-datepicker__arrow-up--top' : ''
          ],
          style: {
            'opacity': _this.isFocus ? '1' : '0'
          }
        }), _this.$slots.default])
      },
      methods: {
        remove () {
          this.$destroy()
          this.hide()
          setTimeout(() => {
            document.body.removeChild(this.$el)
          }, 200)

          //  remove position fixer
          window.removeEventListener('resize', _this.setListPosition)
          document.removeEventListener('scroll', _this.setListPosition, true)
        },
        //  very small a bug with position change: animation
        show () {
          this.$el.style.transformOrigin = _this.arrowTop ? 'bottom' : 'top'
          this.$el.style.transform = 'rotateX(0)'
          this.$el.style.opacity = 1
          _this.setListPosition()
        },
        hide () {
          this.$el.style.transformOrigin = _this.arrowTop ? 'bottom' : 'top'
          this.$el.style.opacity = 0
          this.$el.style.transform = 'rotateX(90deg)'
          _this.setListPosition()
        }
      }
    })

    let dropComponent = DropMenu.$mount()
    let list = dropComponent.$el

    this.list = list

    //  set created transition

    document.body.appendChild(list)
    dropComponent.hide()

    window.addEventListener('resize', this.setListPosition)
    document.addEventListener('scroll', this.setListPosition, true)

    setTimeout(() => {
      dropComponent.show()
    })

    this.action(parent, 'date-picker-drop-menu-register', dropComponent)
  },
  methods: {
    setListPosition () {
      const parent = this.select

      if (parent.isFocus) {
        let parentViewLeft = this.getElementViewLeft(parent.$el)
        let parentViewTop = this.getElementViewTop(parent.$el)

        let parentOffsetHeight = parent.$el.offsetHeight
        let parentOffsetWidth = parent.$el.offsetWidth
        let windowViewHeight = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight

        let listLeft = parentViewLeft
        let listTop = null

        let listWidth = this.list.offsetWidth
        let listHeight = this.list.offsetHeight

        if (listWidth + parentViewLeft >= document.body.offsetWidth) {
          listLeft -= listWidth - parentOffsetWidth
          this.arrowRight = true
        } else {
          this.arrowRight = false
        }

        if (parentViewTop + listHeight + parentOffsetHeight + 5 > windowViewHeight) {
          this.arrowTop = true
          listTop = parentViewTop - listHeight - 5
        } else {
          this.arrowTop = false
          listTop = parentViewTop + parentOffsetHeight + 5
        }

        this.list.style.left = `${listLeft}px`
        this.list.style.top = `${listTop}px`

        DropMenu.$forceUpdate()
      }
    }
  },
  beforeDestroy () {
    DropMenu.remove()
  }
}
</script>

<style scoped>

</style>
