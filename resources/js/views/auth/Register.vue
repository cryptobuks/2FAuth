<template>
    <form-wrapper :title="$t('auth.register')" :fail="fail" :success="success">
        <form @submit.prevent="handleSubmit" @keydown="form.onKeydown($event)">
            <form-field :form="form" fieldName="name" inputType="text" :label="$t('auth.forms.name')" autofocus />
            <form-field :form="form" fieldName="email" inputType="email" :label="$t('auth.forms.email')" />
            <form-field :form="form" fieldName="password" inputType="password" :label="$t('auth.forms.password')" />
            <form-field :form="form" fieldName="password_confirmation" inputType="password" :label="$t('auth.forms.confirm_password')" />
            <form-buttons :isBusy="form.isBusy" :isDisabled="form.isDisabled" :caption="$t('auth.register')" />
        </form>
        <p>{{ $t('auth.forms.already_register') }}&nbsp;<router-link :to="{ name: 'login' }" class="is-link">{{ $t('auth.sign_in') }}</router-link></p>
    </form-wrapper>
</template>

<script>

    import Form from './../../components/Form'

    export default {
        data(){
            return {
                success: '',
                fail: '',
                form: new Form({
                    name : '',
                    email : '',
                    password : '',
                    password_confirmation : '',
                })
            }
        },

        methods : {
            async handleSubmit(e) {
                e.preventDefault()

                const { data } = await this.form.post('api/register')

                if( this.form.errors.any() === false ) {

                    localStorage.setItem('user',data.message.name)
                    localStorage.setItem('jwt',data.message.token)

                    if (localStorage.getItem('jwt') != null) {
                        this.$router.push({name: 'accounts'})
                    }
                }
            }
        },

        beforeRouteEnter (to, from, next) {
            if (localStorage.getItem('jwt')) {
                return next('/');
            }

            next(async vm => {
                const { data } = await vm.axios.post('api/checkuser')

                if( data.userCount > 0 ) {
                    vm.form.isDisabled = true
                    vm.fail = vm.$t('errors.already_one_user_registered') + ' ' + vm.$t('errors.cannot_register_more_user')
                }
            });
        }
    }
</script>