<style scoped>
    .ui.accordion.menu a.title.item {
        padding: 0.92857143em 1.14285714em;
        background: rgba(0, 0, 0, 0.05);
    }

    .ui.fluid.card {
        cursor: pointer;
    }

    .scrolling.undetached.dimmable.dimmed > .dimmer {
        overflow: scroll !important;
    }

    .scrolling.undetached.dimmable.dimmed{
        overflow: hidden !important;
    }
    .save-filter-popup {
        width: 350px;
        max-width: 350px;
    }

    .faded {
        opacity: 0.2;
        transition: all .4s ease;
    }

    .extra.content .label {
        margin-bottom:2px
    }

</style>

<template>
    <span>
        <div v-if="show_picker_button">
            <strong>
                <a @click="show">Croudie Picker</a>
            </strong>
        </div>
        <div v-if="show_selected && selected.length">
            <span v-for="croudie in selected">
                <img class="ui mini circular inline image" :title="croudie.name" :src="croudie.avatar">
            </span>
        </div>
        <semantic-modal size="large" :scrolling="true" :active.sync="showModal" :settings="pickerModalSettings">
            <div v-if="refreshFilters"></div>
            <div class="ui basic segment">
                <div class="ui top aligned two column grid">
                    <div class="column">
                        <h2 class="ui header">Croudie Picker <small v-show="filteredCroudies.length > 1">({{ meta.pagination.total }} found)</small></h2>
                    </div>
                    <div class="right floated right aligned column">
                        <a v-if="filteredCroudies.length > 1 && filteredCroudies.length < 50" transition="fade" class="ui blue basic button" @click="addAll">Select all {{ filteredCroudies.length }} croudies</a>
                        <a v-el:save_filter_button class="ui blue basic button" @click.prevent>Save Filter</a>
                        <div v-el:save_filter_popup class="ui custom popup text-left save-filter-popup">
                            <div class="header">Save this filter</div>
                            <div class="content">

                                <div class="item well well-blank well-small" v-if="currentfilter && currentfilter.user.data.code != current_user.code" class="ui segment basic">
                                    This filter was created by {{currentfilter.user.data.name}}. When saved a copy will be created.
                                </div>

                                <div class="item well well-blank well-small">
                                    <input class="textbox block" placeholder="Filter name.." v-model="new_filter_name">
                                </div>

                                <div class="column">
                                    <a href="#" class="actionLink small light push-left" @click.prevent="cancelFilter()">Cancel</a>
                                </div>

                                <div class="column text-right">
                                    <a href="#" v-show="currentfilter && currentfilter.user.data.code == current_user.code" class="actionLink small alt" @click.prevent="updateFilter()">Update Filter</a>

                                    <a href="#" class="actionLink small yellow" @click.prevent="saveFilter()">Save As New</a>
                                </div>
                            </div>
                        </div>
                        <a class="ui blue basic button" @click="showModal = false">Continue</a>
                    </div>
                </div>
            </div>
            <div :class="filterClasses">
                <div class="ui five wide secondary column">
                    <div class="ui basic segment">
                        <div v-el:accordion class="ui vertical accordion fluid menu">
                            <a class="active title item">
                                <i class="dropdown icon"></i>
                                Rules
                            </a>
                            <div class="active content">
                                <croudie-picker-menu
                                :croudie.sync="croudie"
                                :client.sync="client"
                                :online.sync="online"
                                :date-filter.sync="dateFilter"
                                :language.sync="language"
                                :country.sync="country"
                                :qualification.sync="qualification"
                                :availability.sync="availability"
                                :rate.sync="rate"
                                :search.sync="search"
                                :rate-histogram="meta.rates">
                            </croudie-picker-menu>
                            </div>

                            <a class="title item">
                                <i class="dropdown icon"></i>
                                Filters
                            </a>
                            <div class="content">
                                <filter-menu
                                :current_user="current_user"
                                :show_filter_popup.sync="show_filter_popup"
                                :currentfilter.sync="currentfilter"
                                :rules.sync="filters"
                                :selected_users.sync="selected">
                            </filter-menu>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="ui seven wide center aligned column">
                    <div v-if="filteredCroudies.length === 0 && croudies.length > 0" class="ui very padded basic segment">
                        <h4 class="ui icon grey header">
                            <i class="circular users icon"></i>
                            All croudies in this filter have been selected
                        </h4>
                    </div>
                    <div v-if="croudies.length === 0" class="ui very padded basic segment">
                        <h4 class="ui icon grey header">
                            <i class="circular users icon"></i>
                            No croudies match this filter
                        </h4>
                    </div>
                    <div :class="dimmerClasses">
                        <div class="ui text large loader">Loading</div>
                    </div>
                    <div v-for="croudie in filteredCroudies | limitBy limit" @click="add(croudie)" class="ui fluid grey card" >
                        <div class="left aligned content">
                            <img v-if="croudie.avatar" class="left floated mini circular ui image" :src="croudie.avatar" />
                            <span v-if="croudie.system" class="right floated time">
                                <div title="Supercroud" class="ui circular basic blue label">S</div>
                            </span>
                            <span v-if="croudie.rate" class="right floated time">
                                <div class="ui basic blue label">{{ croudie.rate | currency '£' }}</div>
                            </span>
                            <div class="header">{{ croudie.name }}</div>
                            <div class="meta">{{ croudie.email }}</div>
                        </div>
                        <div class="extra content right aligned" v-if="croudie.languages.data.length ">
                            <div class="ui basic label tiny" v-for="language in croudie.languages.data" track-by="$index">{{language}}</div>
                        </div>
                        <div class="ui inverted indicating bottom attached progress" :data-percent="Math.min(100, (croudie.score / meta.max) * 100)" :title="Math.round(croudie.minutes / 60) + ' hours worked this month'">
                            <div class="bar"></div>
                        </div>
                        <!-- <div class="extra content right aligned" v-if="croudie.qualifications.data.length ">
                            <div class="ui basic label blue tiny" v-for="qualification in croudie.qualifications.data" track-by="$index">{{qualification}}</div>
                        </div> -->
                    </div>
                    <a v-if="filteredCroudies.length > limit || meta.pagination.current_page < meta.pagination.total_pages" class="ui circular basic icon button" @click="loadMore">
                        Show more
                        <semantic-icon class="dropdown"></semantic-icon>
                    </a>
                </div>

                <div class="ui four wide column">
                    <div class="ui basic segment">
                        <strong class="ui header">Selected Croudies <small v-if="selected.length" @click.stop="selected = []">clear</small></strong>
                        <div class="ui relaxed list">
                            <div class="item" v-for="croudie in selected">
                                <img v-if="croudie.avatar" class="left floated mini circular ui image" :src="croudie.avatar" />
                                {{ croudie.name }}
                                <semantic-icon icon="close" colour="red" link @click="remove(croudie)"></semantic-icon>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </semantic-modal>
    </span>
</template>

<script>
    import Vue from 'vue'
    import _ from 'underscore'
    import moment from 'moment'
    import FilterMenu from './FilterMenu.vue'
    import CroudiePickerMenu from './CroudiePickerMenu.vue'

    export default {
        props: {
            show_picker_button : {
                default() {
                    return true
                },
            },
            show_selected: {
                default() {
                    return false
                },
            },

            selected: {
                default() {
                    return []
                },
            },
            rawFilters: {
                default() {
                    return {}
                },
            },

            language: {
                default() {
                    return []
                },
            },
            country: {
                default() {
                    return []
                },
            },
            qualification: {
                default() {
                    return []
                },
            },
            client: {
                default() {
                    return []
                },
            },
            availability: {
                default() {
                    return []
                },
            },
            croudie: {
                default() {
                    return null
                },
            },
            current_user: {
                default() {
                    return null
                }
            },
            online: {
                default() {
                    return null
                }
            },
            dateFilter: {
                default() {
                    return 'last_login'
                }
            },

            frontEndFiltering: {
                default() {
                    return false
                }
            }
        },

        data() {
            return {
                meta: {
                    score: 1,
                    pagination: {
                        total: 1,
                    },
                },
                pickerModalSettings: {
                  closeable: true,
                  closeable_button: true,
                  onHidden: () => {
                    this.$emit('picker-closed')
                  }
                },
                rateHistogram: [],
                croudies: [],
                loading: true,
                showModal: false,
                currentfilter : null,
                search: '',
                rate: 15,
                limit: 5,
                hourLimit: 29,
                show_filter_popup : false,
                new_filter_name: null,
                new_filter_users: false,
                refresh: _.debounce((data) => {
                    this.loading = true
                    this.$http.post('api/user/pick', data).then((response) => {
                        this.$set('croudies', response.data.data)
                        this.$set('meta', response.data.meta)
                        this.loading = false
                        this.limit = 5
                        this.$nextTick(() => {
                            this.$broadcast('refresh-modal')
                            $('.progress').progress()
                        })
                    })
               }, 50),
            }
        },

        ready() {

            let inst = this

            $(this.$els.accordion).accordion()
            $.fn['iCheck'] = function() {}

            $(this.$els.save_filter_button).popup({
                popup: $(this.$els.save_filter_popup),
                on: 'click',
                exclusive: true,
                inline: false,
                position: 'bottom right',
                closable: false,
                onShow : function(){
                    inst.show_filter_popup = true
                },
                onHide : function() {

                    inst.show_filter_popup = false
                }

            });

        },

        components: {
            FilterMenu,
            CroudiePickerMenu,
        },

        methods: {
            show() {
                this.showModal = true
                $('.progress').progress()
            },

            saveFilter() {
                this.$broadcast('save-filter', {
                    new_filter_name : this.new_filter_name,
                    new_filter_users : this.new_filter_users,
                    record_type : 'new'
                })
            },

            updateFilter() {
                this.$broadcast('save-filter', {
                    new_filter_name : this.new_filter_name,
                    new_filter_users : this.new_filter_users,
                    record_type : 'update'
                })
            },

            cancelFilter() {
                $(this.$els.save_filter_button).popup('hide')
            },

            add(croudie) {
                this.selected.push(croudie)
                this.$http.post('api/analytics', {
                    data: {
                        search: this.filters,
                        user_id: croudie.id,
                        user_name: croudie.name,
                        pick: 1,
                    },
                    type: 'croudie_pick',
                })
            },

            remove(croudie) {
                this.selected.$remove(croudie)
            },

            loadMore() {
                this.limit += 5
                this.$emit('load-more')
                this.$nextTick(() => {
                    this.$broadcast('refresh-modal')
                    $('.progress').progress()
                })
            },

            hideSelected(data) {
                return data.filter(croudie => {
                    return this.selected.map((s) => s.code).indexOf(croudie.code) === -1
                })
            },

            addAll() {
                this.filteredCroudies.map(croudie => {
                    this.add(croudie)
                })
            },
        },

        computed: {

            filterClasses() {

                let classNames = ['ui stackable grid']

                if (this.show_filter_popup)
                    classNames.push('faded')

                return classNames.join(' ')

            },

            filteredCroudies() {

                if (!this.croudies || !this.croudies.length) return []

                this.$nextTick(() => {
                    $('.progress').progress()
                })

                //const filterBy = Vue.filter('filterBy')
                //return this.hideSelected(this.croudies)
                return this.hideSelected(this.croudies.filter(croudie => {
                    if (!this.frontEndFiltering) {
                        return true
                    }

                    return (!this.language.length || (croudie.languages.data.length && this.language.some(language => {
                        return croudie.languages.data.indexOf(language.name) !== -1
                    })))

                    && (!this.country.length || (croudie.countries.data.length && this.country.some(country => {
                        return croudie.countries.data.indexOf(country.name) !== -1
                    })))

                    && (!this.qualification.length || (croudie.qualifications.data.length && this.qualification.some(qualification => {
                        return croudie.qualifications.data.indexOf(qualification.name) !== -1
                    })))

                    && (!this.availability.length || (croudie.availability.data.length && this.availability.some(availability => {
                        return croudie.availability.data.indexOf(availability) !== -1
                    })))

                    && ((!this.online || this.croudie !== 0) || moment().subtract(1, this.online).isBefore(croudie[this.dateFilter]))
                    && (this.croudie === null || croudie.system === this.croudie)
                    && (this.rate === '0' || croudie.rate <= this.rate)
                }))
            },

            refreshFilters() {
                this.refresh(this.filters)
            },

            filters() {
                return this.frontEndFiltering ? {
                    clients: this.client.map(client => client.id),
                } : {
                    languages: this.language.map((language) => language.id),
                    countries: this.country.map((country) => country.id),
                    qualifications: this.qualification.map((qualification) => qualification.id),
                    clients: this.client.map(client => client.id),
                    availability: this.availability,
                    tags: this.tags,
                    system: this.croudie,
                    rate: this.rate,
                    online: this.online,
                    search: this.search,
                }
            },

            dimmerClasses() {
                return {
                    ui: true,
                    active: this.loading,
                    inverted:true,
                    dimmer: true,
                }
            },
        },

        watch: {
            croudie(value) {
                if (value === 0 && parseFloat(this.rate, 10) < 1) {
                    this.rate = 15
                }
            }
        },

        events: {
            'close-save-dialog'() {
                this.cancelFilter()
            },
            'open-croudie-picker'() {
                this.show()
            },
            'filter-change'(filter) {

                this.currentfilter = filter
                this.new_filter_name = filter.name
                this.selected = filter.rules.data.length > 0 ? filter.rules.data[0].users.data : []
                this.language = filter.rules.data.length > 0 ? filter.rules.data[0].languages.data : []
                this.country = filter.rules.data.length > 0 ? filter.rules.data[0].countries.data : []
                this.qualification = filter.rules.data.length > 0 ? filter.rules.data[0].qualifications.data : []
                this.availability = filter.rules.data.length > 0 ? filter.rules.data[0].availability.data.map(day => day.day_of_week) : []
                this.croudie = filter.rules.data.length > 0 ? filter.rules.data[0].system : null
                this.rate = filter.rules.data > 0 ? parseFloat(filter.rules.data[0].rate, 10) : 15
            },

            'load-more'() {
                const data = this.filters
                data.page = this.meta.pagination.current_page + 1

                this.$http.post('api/user/pick', data).then((response) => {
                    this.croudies.push.apply(this.croudies, response.data.data)
                    this.$set('meta', response.data.meta)
                    this.$nextTick(() => {
                        this.$broadcast('refresh-modal')
                        $('.progress').progress()
                    })
                })
            },
        },
    }
</script>
