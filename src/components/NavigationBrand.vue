<template>
	<div class="brand" @click="toggleBrandMenu">
		<span class="brand__name brand__name--small"><b>A</b>SF</span>
		<span class="brand__name brand__name--big"><b>Archi</b>SteamFarm</span>
		<div v-if="authenticated" class="brand__icon">
			<font-awesome-icon v-if="brandMenu" icon="times"></font-awesome-icon>
			<font-awesome-icon v-else icon="angle-down"></font-awesome-icon>
		</div>

		<transition name="brand__menu">
			<div class="brand__menu" v-if="brandMenu && authenticated">
				<div class="brand__menu-item" @click.stop="update">
					<font-awesome-icon class="brand__menu-icon" icon="cloud-download-alt" fixed-width></font-awesome-icon>
					<span>{{ $t('update') }}</span>
				</div>

				<div class="brand__menu-item" @click.stop="restart">
					<font-awesome-icon class="brand__menu-icon" icon="power-off" fixed-width></font-awesome-icon>
					<span>{{ $t('restart') }}</span>
				</div>

				<div class="brand__menu-item" @click.stop="exit">
					<font-awesome-icon class="brand__menu-icon" icon="sign-out-alt" fixed-width></font-awesome-icon>
					<span>{{ $t('exit') }}</span>
				</div>
			</div>
		</transition>
	</div>
</template>

<script>
	import waitForRestart from '../utils/waitForRestart';

	import { mapGetters } from 'vuex';

	export default {
		name: 'navigation-brand',
		data() {
			return {
				brandMenu: false,
				restarting: false
			};
		},
		computed: mapGetters({
			authenticated: 'auth/authenticated'
		}),
		methods: {
			toggleBrandMenu() {
				this.brandMenu = !this.brandMenu;
			},
			async update() {
				try {
					const response = await this.$http.post('asf/update');
					this.brandMenu = false;

					if (response.Success) {
						await waitForRestart();
						window.location.reload(true);
					}
				} catch (err) {
					this.$error(err.message);
				}
			},
			async restart() {
				if (this.restarting) return;
				this.restarting = true;

				try {
					await this.$http.post('asf/restart');
					this.$info(this.$t('restart-initiated'));
					await waitForRestart();
					this.$success(this.$t('restart-complete'));
					this.brandMenu = false;
				} catch (err) {
					this.$error(err.message);
				} finally {
					this.restarting = false;
				}
			},
			async exit() {
				try {
					await this.$http.post('asf/exit');
					this.$info(this.$t('exit-message'));
					this.brandMenu = false;
				} catch (err) {
					this.$error(err.message);
				}
			}
		}
	};
</script>

<style lang="scss">
	.brand {
		padding: 0 1em;
		background: var(--color-theme-dark);
		width: var(--navigation-width);
		color: var(--color-text);
		box-sizing: border-box;
		display: flex;
		align-items: center;
		justify-content: space-between;
		cursor: pointer;
		height: var(--navigation-height);
		transition: ease-in-out width .3s;
		position: relative;

		.app--not-authorized & {
			cursor: initial;
			justify-content: center;
		}

		.app--small-navigation & {
			padding: 0;
			justify-content: center;
			flex-direction: column;

			.brand__name--big {
				display: none;
			}

			.brand__icon {
				line-height: 0.5;
			}

			.brand__name--small {
				display: block;
			}
		}
	}

	.brand__name {
		font-size: 1.25em;
		overflow: hidden;
	}

	.brand__name--small {
		display: none;
	}

	.brand__menu {
		position: absolute;
		top: var(--navigation-height);
		width: 100%;
		left: 0;
		transition: transform .3s;
		transform-origin: top;
	}

	.brand__menu-item {
		height: var(--navigation-height);
		background: var(--color-theme);
		color: var(--color-text);
		display: flex;
		align-items: center;
		cursor: pointer;
		padding: 0 1.5em;

		.app--small-navigation & {
			padding: 0;
			justify-content: center;
		}

		&:hover {
			background: var(--color-theme-dark);
		}
	}

	.brand__menu-icon {
		margin-right: 0.5em;

		.app--small-navigation & {
			margin-right: 0;

			+ span {
				display: none;
			}
		}
	}

	.brand__menu-enter, .brand__menu-leave-to {
		transform: scaleY(0);
	}
</style>
