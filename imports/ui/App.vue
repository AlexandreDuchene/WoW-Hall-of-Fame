<template>
    <v-app>
        <v-app-bar app dense>
            <v-img src="/img/logo.png" max-height="45px" max-width="45px"></v-img>
            <v-toolbar-title class="font-weight-black red--text text--darken-4 text-uppercase">
                {{ settings.guildName }} - HALL OF FAME
            </v-toolbar-title>

            <v-spacer></v-spacer>

            <v-app-bar-nav-icon color="red darken-4" @click="menuDrawer = true">
                <v-icon>mdi-cog</v-icon>
            </v-app-bar-nav-icon>
        </v-app-bar>

        <v-navigation-drawer v-model="menuDrawer" fixed temporary right>
            <v-list nav dense>
                <v-list-item>
                    <v-list-item-avatar>
                        <v-switch id="switch-theme" @change="setTheme" color="red darken-4"></v-switch>
                    </v-list-item-avatar>
                    <v-list-item-title>
                        <label class="font-weight-black red--text text--darken-4" for="switch-theme">{{ $t('dark-theme') }}</label>
                    </v-list-item-title>
                </v-list-item>
                <v-list-item>
                    <v-list-item-avatar>
                        <a target="_blank" rel="noopener noreferrer" class="font-weight-black red--text text--darken-4" :href="settings.warcraftlogsUrl + 'guild/' + settings.region + '/' + settings.server + '/' + settings.guildName">
                            <v-img src="/img/warcraftlogs.png" max-height="35px" max-width="35px" ></v-img>
                        </a>
                    </v-list-item-avatar>
                    <v-list-item-title>
                        <a target="_blank" rel="noopener noreferrer" class="font-weight-black red--text text--darken-4" :href="settings.warcraftlogsUrl + 'guild/' + settings.region + '/' + settings.server + '/' + settings.guildName">Warcraftlogs</a>
                    </v-list-item-title>
                </v-list-item>
            </v-list>
        </v-navigation-drawer>

        <v-content>
            <v-container fluid grid-list-xl>
                <v-layout wrap justify-space-between>
                    <v-flex xs12 md8>
                        <v-card>
                            <v-tabs v-model="activeTab" color="red darken-4">
                                <v-tab v-for="tab in tabs" :key="tab">
                                    {{ $t(tab) }}
                                </v-tab>
                            </v-tabs>
                            <v-tabs-items v-model="activeTab">
                                <v-tab-item v-for="tab in tabs" :key="tab">
                                    <AchievementVue
                                            v-if="tab === 'achievements'"
                                            v-for="achievement in achievements"
                                            :key="achievement._id"
                                            :achievement="achievement"
                                    />
                                    <CharacterVue
                                            v-if="tab === 'characters'"
                                            v-for="character in characters"
                                            :key="character._id"
                                            :character="character"
                                    />
                                </v-tab-item>
                            </v-tabs-items>
                        </v-card>
                    </v-flex>
                    <v-flex xs12 md4>
                        <LatestAchievements></LatestAchievements>
                    </v-flex>
                </v-layout>
            </v-container>
        </v-content>

        <v-footer app>
        </v-footer>
    </v-app>
</template>

<script>
    import AchievementVue from "../ui/Achievement"
    import CharacterVue from "../ui/Character";
    import LatestAchievements from "../ui/LatestAchievements"
    import { Achievement, Character } from "../../client/main";

    export default {
        components: {
            AchievementVue,
            CharacterVue,
            LatestAchievements,
        },
        methods: {
            setTheme() {
                this.$vuetify.theme.dark = !this.$vuetify.theme.dark;
                this.$cookies.set('darkTheme', this.$vuetify.theme.dark);
            }
        },
        meteor: {
            $subscribe: {
                'achievements': [],
                'characters': [],
            },
            achievements() {
                return Achievement.find(
                    {},
                    {transform: function(achievement) {
                        achievement.characters.sort(function(a, b) {
                            return a.name > b.name ? 1 : -1;
                        });

                        return achievement;
                    }},
                ).fetch();
            },
            characters() {
                return Character.find(
                    {achievementsCount: {$gt: 0}},
                    {
                        sort: {'name': 1},
                        transform: function(character) {
                            character.achievements.sort(function(a, b) {
                                return a.dates[0].date > b.dates[0].date ? -1 : 1;
                            });

                            return character;
                        }
                    }
                ).fetch();
            },
        },
        beforeCreate() {
            if (this.$cookies.get('darkTheme') !== undefined) {
                this.$vuetify.theme.dark = this.$cookies.get('darkTheme') === 'true';
            }
        }
    };
</script>

<style>
    a {
        text-decoration: none;
    }

    label {
        cursor: pointer;
    }
</style>
