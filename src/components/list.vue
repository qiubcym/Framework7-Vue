<script>
  export default {
    beforeDestroy: function () {
      var self = this;
      if (!(self.virtual && self.virtualInit && self.f7VirtualList)) return;
      if (self.f7VirtualList.destroy) self.f7VirtualList.destroy();
    },
    watch: {
      virtualItems: function () {
        // Items Updated
        var self = this;
        if (!(self.virtual && self.virtualInit && self.f7VirtualList)) return;
        self.f7VirtualList.replaceAllItems(self.virtualItems);
      },
    },
    render: function (c) {
      var blockEl, blockChildren;
      var self = this;

      blockChildren = self.grouped ? self.$slots.default : c('ul', {}, self.$slots.default)
      var outOfList = [], ulSlots = [];
      if (self.$slots.default) {
        for (var i = 0; i < self.$slots.default.length; i++) {
          var tag = self.$slots.default[i].tag;
          if (tag && !(tag == 'li' || tag.indexOf('list-item')>=0 || tag.indexOf('list-button')>=0)) {
            outOfList.push(self.$slots.default[i]);
          }
          else {
            ulSlots.push(self.$slots.default[i]);
          }
        }
      }
      blockEl = c(
        self.form ? 'form' : 'div',
        {
          staticClass: 'list-block',
          'class': {
            'inset': self.inset,
            'media-list': self.mediaList,
            'sortable': self.sortable,
            'accordion-list': self.accordion,
            'contacts-block': self.contacts,
            'virtual-list': self.virtual,
            'tab': self.tab,
            'active': self.active,
            'no-hairlines': self.noHairlines,
            'no-hairlines-between': self.noHairlinesBetween
          },
          on: {
            'sortable:open': self.onSortableOpen,
            'sortable:close': self.onSortableClose,
            'sortable:sort': self.onSortableSort
          }
        },
        [
          ulSlots.length > 0 ? [c('ul', {}, ulSlots), outOfList] : outOfList
        ]
      );
      return blockEl;
    },
    props: {
      'inset': Boolean,
      'media-list': Boolean,
      'grouped': Boolean,
      'sortable': Boolean,
      'form': Boolean,
      'label': String,
      'accordion': Boolean,
      'contacts': Boolean,

      'no-hairlines': Boolean,
      'no-hairlines-between': Boolean,

      // Tab
      'tab': Boolean,
      'active': Boolean,

      // Virtual List
      'virtual': Boolean,
      'virtual-init': {
        type: Boolean,
        default: true
      },
      'virtual-items': [Array, Object],
      'virtual-height': [Number, Function],
      'virtual-rows-before': Number,
      'virtual-rows-after': Number,
      'virtual-cols': {
        type: Number,
        default: 1
      },
      'virtual-cache': {
        type: Boolean,
        default: true
      },
      'virtual-filtered-only': {
        type: Boolean,
        default: false
      },
      'virtual-search-by-item': Function,
      'virtual-search-all': Function,
      'virtual-render-item': Function
    },
    methods: {
      onSortableOpen: function (event) {
        this.$emit('sortable:open', event)
      },
      onSortableClose: function (event) {
        this.$emit('sortable:close', event)
      },
      onSortableSort: function (event) {
        this.$emit('sortable:sort', event, event.detail)
      },
      onF7Init: function (f7) {
        var self = this;
        // Init Virtual List
        if (!(self.virtual && self.virtualInit)) return;
        var $$ = self.$$;
        var template = $$(self.$el).find('script').html();
        if (!template && !self.virtualRenderItem) return;
        if (template) template = self.$t7.compile(template);

        self.f7VirtualList = f7.virtualList(self.$el, {
          items: self.virtualItems || [],
          template: template,
          height: self.virtualHeight || undefined,
          cols: self.virtualCols,
          rowsBefore: self.virtualRowsBefore || undefined,
          rowsAfter: self.virtualRowsAfter || undefined,
          showFilteredItemsOnly: self.virtualFilteredOnly,
          searchByItem: self.virtualSearchByItem,
          searchAll: self.virtualSearchAll,
          renderItem: self.virtualRenderItem,
          onItemBeforeInsert: function (list, item) {
            self.$emit('virtualItemBeforeInsert', list, item);
          },
          onBeforeClear: function (list, fragment) {
            self.$emit('virtualBeforeClear', list, fragment);
          },
          onItemsBeforeInsert: function (list, fragment) {
            self.$emit('virtualItemsBeforeInsert', list, fragment);
          },
          onItemsAfterInsert: function (list, fragment) {
            self.$emit('virtualItemsAfterInsert', list, fragment);
          },
        })
      }
    }
  }
</script>