<template>
    <TemplateRoot class="root">
        <Breadcrumbs :links="breadcrumbs" class="breadcrumbs"/>
        <div class="filters">
            <div class="pagination">
                <span>
                    Сторінка
                    <strong>{{ usersTotal > 0 ? page : 0 }} / {{ countOfPages }}</strong>
                </span>
                <div class="num-inputs-wrapper">
                    <NumberInput
                        :min="1"
                        :initial-value="page"
                        placeholder="Cторінка"
                        label="Cторінка"
                        @end="changePage"
                        class="page-input"
                    />
                    <NumberInput
                        :min="1"
                        :max="100"
                        placeholder="Розмір сторінки"
                        label="Розмір сторінки"
                        @end="changeTake"
                        class="take-input"
                    />
                </div>
            </div>
            <div class="text-inputs-wrapper">
                <TextInput
                    placeholder="Пошук по імені"
                    label="Пошук по імені"
                    @end="changeName"
                />
            </div>
        </div>
        <Card class="main-block">
            <AccentTable
                :columns="tableColumns"
                :rows="tableRows"
            />
        </Card>
    </TemplateRoot>
</template>

<script lang="ts">
import {defineComponent} from 'vue';
import TemplateRoot from '@/components/common/template-root.vue';
import {ActionLog} from '@/interfaces/models/action-log';
import {UserRole} from '@/enums/user-role';
import {TableColumn} from '@/interfaces/table/table-column';
import {TableRow} from '@/interfaces/table/table-row';
import {TableAccent} from '@/enums/table-accent';
import Card from '@/components/block/card.vue';
import AccentTable from '@/components/block/accent-table/accent-table.vue';
import NumberInput from '@/components/forms/number-input.vue';
import TextInput from '@/components/forms/text-input.vue';
import Breadcrumbs from '@/components/common/breadcrumbs.vue';
import {BreadcrumbLink} from '@/interfaces/breadcrumbs-link';

const INITIAL_STEP = 10;

export default defineComponent({
    name: 'UsersAct',
    components: {
        Breadcrumbs,
        TextInput,
        NumberInput,
        AccentTable,
        Card,
        TemplateRoot,
    },
    data() {
        const tableColumns: TableColumn[] = [
            {
                title: 'Користувач',
                key: 'login',
            },
            {
                title: 'ПІБ',
                key: 'name',
            },
        ];

        const breadcrumbs: BreadcrumbLink[] = [
            {
                url: '/registrator',
                text: 'Головна',
            },
            {
                url: '/user-status-list',
                text: 'Список Користувачів',
            },
        ];

        return {
            page: 1,
            take: INITIAL_STEP,
            name: null as string | null,
            role: null as UserRole | null,
            type: null as string | null,
            tableColumns,
            loading: false,
            breadcrumbs,
        };
    },
    computed: {
        users(): ActionLog[] {
            return this.$store.getters['civil-acts/list'];
        },
        usersTotal(): number {
            return this.$store.getters['civil-acts/total'];
        },
        authToken(): string {
            return this.$store.getters['auth/userToken'];
        },
        tableRows(): TableRow[] {
            return this.users.map(({...data}) => ({
                ...data,
                accent: TableAccent.INFO,
                handler: ({login}) => {
                    console.log(login);
                    this.$router.push({name: 'UserActs', params: {login}});
                },
            }));
        },
        countOfPages(): number {
            return Math.ceil(this.usersTotal / this.take);
        },
    },
    methods: {
        fetchLogs(): Promise<void> {
            this.loading = true;

            return this.$store.dispatch('civil-acts/fetchList', {
                authToken: this.authToken,
                take: this.take,
                skip: this.take * (this.page - 1),
                name: this.name ? this.name : undefined,
            })
                .finally(() => {
                    this.loading = false;
                });
        },
        changeTake(take: number) {
            if (take) {
                this.take = take;
                this.page = 1;
                this.fetchLogs();
            }
        },
        changeName(name: string) {
            this.name = name;
            this.fetchLogs();
        },
        changePage(page: number) {
            if (page <= this.countOfPages) {
                this.page = page;
                this.fetchLogs();
            }
        },
    },
    created() {
        if (!this.users.length) {
            this.fetchLogs();
        }
    },
});

</script>

<style scoped lang="scss">
@import 'src/assets/colors';

.root {
    flex-direction : column;

    .breadcrumbs {
        margin-bottom : 20px;
    }

    .main-block {
        position        : relative;
        justify-content : center;
        align-items     : center;
        width           : 100%;
        border          : 1px solid $primary;
        padding         : 0;
        min-height      : 600px;
        overflow-x      : hidden;
    }

    .filters {
        display         : flex;
        justify-content : space-between;
        align-items     : flex-end;
        margin-bottom   : 20px;

        .select-input {
            flex       : 0 0 45%;
            max-height : 60px;
            padding    : 8px 10px;
        }

        .text-inputs-wrapper {
            flex           : 0 0 48%;
            display        : flex;
            flex-direction : column;
        }

        .pagination {
            display         : flex;
            justify-content : space-between;
            flex-wrap       : wrap;
            flex            : 0 0 50%;

            span {
                flex : 0 0 50%;

                strong {
                    font-weight : 600;
                }
            }

            .num-inputs-wrapper {
                flex            : 0 0 100%;
                display         : flex;
                justify-content : space-between;

                .page-input {
                    flex : 0 0 35%;
                }

                .take-input {
                    flex : 0 0 63%;
                }
            }
        }
    }
}

</style>
