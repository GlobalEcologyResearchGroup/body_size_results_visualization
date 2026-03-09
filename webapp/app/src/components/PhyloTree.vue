<template lang="pug">
div
  tree.tree(
    :data="treeData" 
    layoutType="radial"
    type="cluster"
    :duration="50"
    nodeText="name"
    :margin-x="0"
    :margin-y="0"
    :person="selectedPerson"
    v-on:clicked="clicked"
    v-on:hover="hover"
    data-step="2"
    data-intro="This is a family-level phylogeny. You can click on the names at each tip of the tree to learn more about each family and our model results."
    )

  .ui.card.floating-card(v-if="hoveredNode" :style="cardStyle" ref="floatingCard")
    .image(v-if="hoveredImage")
      img(:src="hoveredImage")
    .content
      .header {{ hoveredNode.name }}
      .meta {{ hoveredTaxonomy }}
      .description(v-if="hoveredSummary") {{ hoveredSummary }}

  .ui.raised.container.segment
    my-header(v-bind:main="selectedLeaf ? selectedLeaf : 'Select a Node to View Summary'", 
              v-bind:sub="common")
    .ui.divider(v-if="selectedTree === 'main'")
    table.ui.center.aligned.single.line.table(v-if="selectedTree === 'main'")
      thead
        tr
          th Lower Bound (95% CRI)
          th Mean Estimated Effect
          th Upper Bound (95% CRI)
      tbody
        tr
          td {{ people.lwr_95 }}
          td {{ people.estimate }}
          td {{ people.upr_95 }}
    .ui.horizontal.divider(v-if="selectedLeaf")#summary Summary
    wiki-summary(:selected="selectedLeaf")
  attribution
</template>

<script>
// Components
import {tree} from './vue-d3-tree'
import WikiSummary from './WikiSummary'
import attribution from './Attribution'
import myHeader from './Header'
import wiki from 'wikijs'
const wikiApi = wiki({apiUrl: 'https://en.wikipedia.org/w/api.php'})
// Tree data
import rawTrees from 'tree/trees'
// think of a better name
var parseNewick = require('./parseNewick')

var trees = {}
Object.keys(rawTrees).forEach(x => {
  trees[x] = parseNewick.parseNewick(rawTrees[x])
})

export default {
  name: 'phylo-tree',
  data () {
    return {
      treeData: trees.main,
      selectedPerson: 'main',
      selectedLeaf: '',
      people: {
        lwr_95: 0,
        estimate: 0,
        upr_95: 0
      },
      selectedTree: 'main',
      seen: parseNewick.seen,
      hoveredNode: null,
      hoveredImage: null,
      hoveredSummary: null,
      hoveredTaxonomy: '',
      mouseX: 0,
      mouseY: 0,
      hoverTimeout: null
    }
  },
  methods: {
    hover: function (event) {
      if (this.hoverTimeout) {
        clearTimeout(this.hoverTimeout)
        this.hoverTimeout = null
      }

      if (event.active) {
        this.mouseX = event.event.clientX
        this.mouseY = event.event.clientY
        
        // Show text immediately
        this.hoveredNode = event.data
        this.hoveredImage = null
        this.hoveredSummary = null
        this.updateHoverTaxonomy(event.data.name)
        
        // Delay Wikipedia API requests
        this.hoverTimeout = setTimeout(() => {
          this.updateHoverWiki(event.data.name)
        }, 500)
      } else {
        this.hoveredNode = null
        this.hoveredImage = null
        this.hoveredSummary = null
        this.hoveredTaxonomy = ''
      }
    },
    updateHoverTaxonomy (name) {
      if (!name || name.match(/ott/)) return
      
      const d = this.seen[name]
      if (d) {
        const match = d.Family ? d.Family.match(/\((.*)\)/) : null
        const common = match ? match[1] : ''
        this.hoveredTaxonomy = `Order: ${d.Order}${common ? ' | Common: ' + common : ''}`
      }
    },
    updateHoverWiki (name) {
      if (!name || name.match(/ott/)) return

      wikiApi.page(name).then(page => {
        Promise.all([page.mainImage(), page.summary()]).then(([image, summary]) => {
          if (this.hoveredNode && this.hoveredNode.name === name) {
            this.hoveredImage = image
            this.hoveredSummary = summary.length > 150 ? summary.substring(0, 150) + '...' : summary
          }
        })
      }).catch(() => {})
    },
    updateHoverInfo (name) {
      // Deprecated in favor of split taxonomy/wiki updates
    },
    clicked: function (x) {
      const name = x.data.name
      var d = this.seen[name]
      if (d) {
        this.people.lwr_95 = d.lwr_95
        this.people.estimate = d.estimate
        this.people.upr_95 = d.upr_95
      } else {
        // Reset or handle missing data for orders/groups
        this.people.lwr_95 = 0
        this.people.estimate = 0
        this.people.upr_95 = 0
      }

      this.$scrollTo('#summary')

      this.selectedLeaf = name
    },
    notNullColor (d) {
      return x => '#F00'
      // d3.scaleOrdinal()
         //  .domain(['Bacteria', 'Eukaryota', 'Archaea'])
         //  .range(d3.schemeCategory10)
    }
  },
  watch: {
    selectedTree (newValue) {
      this.selectedLeaf = ''
      this.treeData = trees[newValue]
    }
  },
  computed: {
    common () {
      var d = this.seen[this.selectedLeaf]
      if (d && d.Family) {
        var match = d.Family.match(/\((.*)\)/)
        var c = match ? match[1] : d.Family
        return 'Higher Taxonomy: ' + d.Order + ' | Common: ' + c
      }
      return ''
    },
    cardStyle () {
      const xOffset = 20
      const yOffset = 20
      const margin = 10
      let top = this.mouseY + yOffset
      
      // If card would overflow bottom, shift it up
      if (this.$refs.floatingCard) {
        const height = this.$refs.floatingCard.offsetHeight
        if (top + height > window.innerHeight - margin) {
          top = window.innerHeight - height - margin
        }
      }

      return {
        left: `${this.mouseX + xOffset}px`,
        top: `${top}px`,
        position: 'fixed',
        zIndex: 1000,
        pointerEvents: 'none',
        width: '250px'
      }
    }
  },
  components: {
    tree,
    attribution,
    WikiSummary,
    myHeader
  }
}
</script>

<style>
label {
  padding-right: 1em;
}
.tree {
  margin: 0 auto;
  height: 1100px;
  width: 1300px;
}

@media (max-width: 600px) {
  .tree {
    height: 400px;
    width: 400px;
  }
}
.treeclass .nodetree circle {
  r: 3;
}

.treeclass .node--internal circle {
  r: 0;
}

.link-active {
  opacity: 1;
  stroke: #000;
  stroke-width: 2px !important;
}
.treeclass .nodetree text {
  font: 8px sans-serif;
  cursor: pointer;
  letter-spacing: 0.5px;
  fill: var(--text-color) !important;
}
.treeclass .nodetree text:hover {
  font-weight: bold;
}


.treeclass .nodetree.selected text {
  font-weight: bold;
}

.treeclass .linktree {
  fill: none;
  stroke: #333;
  stroke-opacity: 1;
  stroke-width: 0.5px;
}

th {
  cursor: pointer !important;
}

.positive {
  color: #acdfb7 !important;
}
</style>
