<style scoped>
.parcoords { height: 250px; }

</style>

<template>
    <div class='parcoords' :width='width' v-once>
    </div>
</template>


<script lang='coffee'>

require("./lib/d3.parcoords.js")
{ calc_extent } = require("./normalize.coffee")

module.exports =
    name: 'parallel-coord'
    props:
        data: null
        width: null
        filter: null
        filterChanged: null
        colour: null
        dimensions: null
        highlight: null
    watch:
        dimensions: () ->
            this.update_data()
        data: () ->
            this.update_data()
        filterChanged: () ->
            @parcoords.brush()

        highlight: (d) ->
            if d.length>0
                @parcoords.highlight(d)
            else
                @parcoords.unhighlight()

    mounted: () ->
        @parcoords = d3.parcoords()(this.$el)
            .alpha(0.4)
            .reorderable()
            .brushable() # enable brushing
            .mode('queue')
        @parcoords.setFilter(this.filter)
        @parcoords.on('brush', (d) => this.$emit('brush', d))
        @parcoords.on('render', () => this.$emit('render'))
        this.update_data()

    methods:
        update_data: () ->
            data = this.data
            dims = this.dimensions
            colouring = this.colour

            dim_names = {}
            dims.forEach((k) -> dim_names[k.idx] = k.name)
            @parcoords.data(data)
                 .dimensions(dims.map((c) -> c.idx))
                 .dimension_names(dim_names)
                 .autoscale()
            @parcoords.color(colouring) if colouring?

            extent = calc_extent(data,dims)

            h = @parcoords.height() - @parcoords.margin().top - @parcoords.margin().bottom;
            @parcoords.dimensions().forEach (k) =>
                @parcoords.yscale[k] = d3.scale.linear().domain(extent).range([h+1,1])

            @parcoords.createAxes().render().reorderable().brushable()

            @parcoords.render()
            @parcoords.brush()   # Reset any brushes that were in place

        # These methods just pass through
        # dimensions: () -> @parcoords.dimensions()
        # dimension_names: () -> @parcoords.dimension_names()
        # on: (t,func) -> @parcoords.on(t,func)        # TODO - better abstraction for this
        # brush: () -> @parcoords.brush()

</script>
