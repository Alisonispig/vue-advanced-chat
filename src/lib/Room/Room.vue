<template>
	<div
		v-show="(isMobile && !showRoomsList) || !isMobile || singleRoom"
		class="vac-col-messages"
		@touchstart="touchStart"
	>
		<slot v-if="showNoRoom" name="no-room-selected">
			<div class="vac-container-center vac-room-empty">
				<div>{{ textMessages.ROOM_EMPTY }}</div>
			</div>
		</slot>

		<room-header
			v-else
			:current-user-id="currentUserId"
			:text-messages="textMessages"
			:single-room="singleRoom"
			:show-rooms-list="showRoomsList"
			:is-mobile="isMobile"
			:room-info-enabled="roomInfoEnabled"
			:menu-actions="menuActions"
			:room="room"
			:message-selection-enabled="messageSelectionEnabled"
			:message-selection-actions="messageSelectionActions"
			:selected-messages-total="selectedMessages.length"
			@toggle-rooms-list="$emit('toggle-rooms-list')"
			@room-info="$emit('room-info')"
			@menu-action-handler="$emit('menu-action-handler', $event)"
			@message-selection-action-handler="messageSelectionActionHandler"
			@cancel-message-selection="messageSelectionEnabled = false"
		>
			<template v-for="(i, name) in $scopedSlots" #[name]="data">
				<slot :name="name" v-bind="data" />
			</template>
		</room-header>

		<div
			id="messages-list"
			class="vac-container-scroll"
		>
      <virtual-list
        ref="scrollContainer"
        style="height: 100%;overflow-y: auto;"
        class="vac-messages-container"
        :top-threshold="scrollDistance"
        :data-key="'_id'"
        :data-sources="messages"
        :data-component="RoomMessage"
        :extra-props="{
              currentUserId:currentUserId,
              messages:messages,
              editedMessageId:editedMessageId,
              messageActions:messageActions,
              roomUsers: room.users,
              textMessages: textMessages,
              newMessages:newMessages,
              showReactionEmojis: showReactionEmojis,
              showNewMessagesDivider: showNewMessagesDivider,
              textFormatting: textFormatting,
              linkOptions: linkOptions,
              usernameOptions: usernameOptions,
              messageSelectionEnabled: messageSelectionEnabled,
              selectedMessages: selectedMessages,
              openFile: openFile,
              openUserTag: openUserTag,
              sendMessageReaction: sendMessageReaction,
              unselectMessage: unselectMessage,
              onMessageAdded: onMessageAdded,
              selectMessage: selectMessage,
              messageActionHandler: messageActionHandler,
            }"
        @totop="loadMoreMessages"
        @scroll="onContainerScroll"
      >
        <template #header>
          <loader :show="loadingMessages">
            <template v-for="(idx, name) in $scopedSlots" #[name]="data">
              <slot :name="name" v-bind="data" />
            </template>
          </loader>
          <transition name="vac-fade-message">
            <div>
              <div v-if="showNoMessages" class="vac-text-started">
                <slot name="messages-empty">
                  {{ textMessages.MESSAGES_EMPTY }}
                </slot>
              </div>
              <div v-if="showMessagesStarted" class="vac-text-started">
                {{ textMessages.CONVERSATION_STARTED }} {{ messages[0].date }}
              </div>
            </div>
          </transition>
          <div
            v-if="messages.length && !messagesLoaded"
            id="infinite-loader-messages"
          >
            <loader :show="true" :infinite="true">
              <template v-for="(idx, name) in $scopedSlots" #[name]="data">
                <slot :name="name" v-bind="data" />
              </template>
            </loader>
          </div>
        </template>
      </virtual-list>
		<div v-if="!loadingMessages">
			<transition name="vac-bounce">
				<div v-if="scrollIcon" class="vac-icon-scroll" @click="scrollToBottom">
					<transition name="vac-bounce">
						<div
							v-if="scrollMessagesCount"
							class="vac-badge-counter vac-messages-count"
						>
							{{ scrollMessagesCount }}
						</div>
					</transition>
					<slot name="scroll-icon">
						<svg-icon name="dropdown" param="scroll" />
					</slot>
				</div>
			</transition>
		</div>
    </div>
		<room-footer
			:room="room"
			:room-id="roomId"
			:room-message="roomMessage"
			:text-messages="textMessages"
			:show-send-icon="showSendIcon"
			:show-files="showFiles"
			:show-audio="showAudio"
			:show-emojis="showEmojis"
			:show-footer="showFooter"
			:accepted-files="acceptedFiles"
			:textarea-action-enabled="textareaActionEnabled"
			:textarea-auto-focus="textareaAutoFocus"
			:user-tags-enabled="userTagsEnabled"
			:emojis-suggestion-enabled="emojisSuggestionEnabled"
			:templates-text="templatesText"
			:text-formatting="textFormatting"
			:link-options="linkOptions"
			:audio-bit-rate="audioBitRate"
			:audio-sample-rate="audioSampleRate"
			:init-reply-message="initReplyMessage"
			:init-edit-message="initEditMessage"
			@update-edited-message-id="editedMessageId = $event"
			@edit-message="$emit('edit-message', $event)"
			@send-message="$emit('send-message', $event)"
			@typing-message="$emit('typing-message', $event)"
			@textarea-action-handler="$emit('textarea-action-handler', $event)"
		>
			<template v-for="(idx, name) in $scopedSlots" #[name]="data">
				<slot :name="name" v-bind="data" />
			</template>
		</room-footer>

	</div>
</template>

<script>
import Loader from '../../components/Loader/Loader'
import SvgIcon from '../../components/SvgIcon/SvgIcon'

import RoomHeader from './RoomHeader/RoomHeader'
import RoomFooter from './RoomFooter/RoomFooter'
import RoomMessage from './RoomMessage/RoomMessage'
import VirtualList from 'vue-virtual-scroll-list'

export default {
	name: 'Room',
	components: {
		Loader,
		SvgIcon,
		RoomHeader,
		RoomFooter,
    VirtualList
	},

	props: {
		currentUserId: { type: [String, Number], required: true },
		textMessages: { type: Object, required: true },
		singleRoom: { type: Boolean, required: true },
		showRoomsList: { type: Boolean, required: true },
		isMobile: { type: Boolean, required: true },
		rooms: { type: Array, required: true },
		roomId: { type: [String, Number], required: true },
		loadFirstRoom: { type: Boolean, required: true },
		messages: { type: Array, required: true },
		roomMessage: { type: String, default: null },
		messagesLoaded: { type: Boolean, required: true },
		menuActions: { type: Array, required: true },
		messageActions: { type: Array, required: true },
		messageSelectionActions: { type: Array, required: true },
		autoScroll: { type: Object, required: true },
		showSendIcon: { type: Boolean, required: true },
		showFiles: { type: Boolean, required: true },
		showAudio: { type: Boolean, required: true },
		audioBitRate: { type: Number, required: true },
		audioSampleRate: { type: Number, required: true },
		showEmojis: { type: Boolean, required: true },
		showReactionEmojis: { type: Boolean, required: true },
		showNewMessagesDivider: { type: Boolean, required: true },
		showFooter: { type: Boolean, required: true },
		acceptedFiles: { type: String, required: true },
		textFormatting: { type: Object, required: true },
		linkOptions: { type: Object, required: true },
		loadingRooms: { type: Boolean, required: true },
		roomInfoEnabled: { type: Boolean, required: true },
		textareaActionEnabled: { type: Boolean, required: true },
		textareaAutoFocus: { type: Boolean, required: true },
		userTagsEnabled: { type: Boolean, required: true },
		emojisSuggestionEnabled: { type: Boolean, required: true },
		scrollDistance: { type: Number, required: true },
		templatesText: { type: Array, default: null },
		usernameOptions: { type: Object, required: true }
	},

	emits: [
		'toggle-rooms-list',
		'room-info',
		'menu-action-handler',
		'message-selection-action-handler',
		'edit-message',
		'send-message',
		'delete-message',
		'message-action-handler',
		'fetch-messages',
		'send-message-reaction',
		'typing-message',
		'open-file',
		'open-user-tag',
		'open-failed-message',
		'textarea-action-handler'
	],

	data() {
		return {
			editedMessageId: null,
			initReplyMessage: null,
			initEditMessage: null,
			infiniteState: null,
			loadingMessages: false,
			showLoader: true,
			loadingMoreMessages: false,
			scrollIcon: false,
			scrollMessagesCount: 0,
			newMessages: [],
			messageSelectionEnabled: false,
			selectedMessages: [],
      RoomMessage: RoomMessage
		}
	},

	computed: {
		room() {
			return this.rooms.find(room => room.roomId === this.roomId) || {}
		},
		showNoMessages() {
			return (
				this.roomId &&
				!this.messages.length &&
				!this.loadingMessages &&
				!this.loadingRooms
			)
		},
		showNoRoom() {
			const noRoomSelected =
				(!this.rooms.length && !this.loadingRooms) ||
				(!this.roomId && !this.loadFirstRoom)

			if (noRoomSelected) {
				this.loadingMessages = false /* eslint-disable-line vue/no-side-effects-in-computed-properties */
			}
			return noRoomSelected
		},
		showMessagesStarted() {
			return this.messages.length && this.messagesLoaded
		}
	},

	watch: {
		loadingMessages(val) {
			if (val) {
				this.infiniteState = null
			} else {
				if (this.infiniteState) this.infiniteState.loaded()
			}
		},
		roomId() {
			this.onRoomChanged()
		},
		messages: {
			deep: true,
			handler(newVal, oldVal) {
				newVal.forEach((message, i) => {
					if (this.showNewMessagesDivider && !message.seen && message.senderId !== this.currentUserId) {
						this.newMessages.push({
							_id: message._id,
							index: i
						})
					}
				})
				if (oldVal?.length === newVal?.length - 1) {
					this.newMessages = []
				}
				if (this.infiniteState) {
					this.infiniteState.loaded()
				}
				setTimeout(() => (this.loadingMoreMessages = false))
			}
		},
		messagesLoaded(val) {
			if (val) this.loadingMessages = false
			if (this.infiniteState) this.infiniteState.complete()
		}
	},

	mounted() {
		this.newMessages = []
	},

	methods: {
		touchStart(touchEvent) {
			if (this.singleRoom) return

			if (touchEvent.changedTouches.length === 1) {
				const posXStart = touchEvent.changedTouches[0].clientX
				const posYStart = touchEvent.changedTouches[0].clientY

				addEventListener(
					'touchend',
					touchEvent => this.touchEnd(touchEvent, posXStart, posYStart),
					{ once: true }
				)
			}
		},
		touchEnd(touchEvent, posXStart, posYStart) {
			if (touchEvent.changedTouches.length === 1) {
				const posXEnd = touchEvent.changedTouches[0].clientX
				const posYEnd = touchEvent.changedTouches[0].clientY

				const swippedRight = posXEnd - posXStart > 100
				const swippedVertically = Math.abs(posYEnd - posYStart) > 50

				if (swippedRight && !swippedVertically) {
					this.$emit('toggle-rooms-list')
				}
			}
		},
		onRoomChanged() {
			this.loadingMessages = true
			this.scrollIcon = false
			this.scrollMessagesCount = 0
			this.resetMessageSelection()

			if (!this.messages.length && this.messagesLoaded) {
				this.loadingMessages = false
			}

			const unwatch = this.$watch(
				() => this.messages,
				val => {
					if (!val || !val.length) return

					const element = this.$refs.scrollContainer
					if (!element) return

					unwatch()

					setTimeout(() => {
						this.scrollToBottom()
						this.loadingMessages = false
					})
				}
			)
		},
		resetMessageSelection() {
			this.messageSelectionEnabled = false
			this.selectedMessages = []
		},
		selectMessage(message) {
			this.selectedMessages.push(message)
		},
		unselectMessage(messageId) {
			this.selectedMessages = this.selectedMessages.filter(
				message => message._id !== messageId
			)
		},
		onMessageAdded({ message, index, ref }) {
      console.log('onMessageAdded')
			if (index !== this.messages.length - 1) return

			const autoScrollOffset = ref.offsetHeight + 60

			setTimeout(() => {
				const scrolledUp =
					this.getBottomScroll(this.$refs.scrollContainer) > autoScrollOffset

				if (message.senderId === this.currentUserId) {
					if (scrolledUp) {
						if (this.autoScroll.send.newAfterScrollUp) {
							this.scrollToBottom()
						}
					} else {
						if (this.autoScroll.send.new) {
							this.scrollToBottom()
						}
					}
				} else {
					if (scrolledUp) {
						if (this.autoScroll.receive.newAfterScrollUp) {
							this.scrollToBottom()
						} else {
							this.scrollIcon = true
							this.scrollMessagesCount++
						}
					} else {
						if (this.autoScroll.receive.new) {
							this.scrollToBottom()
						} else {
							this.scrollIcon = true
							this.scrollMessagesCount++
						}
					}
				}
			})
		},
		onContainerScroll(e, range) {
      console.log(e, range)
			if (!e.target) return

			const bottomScroll = this.getBottomScroll(e.target)
			if (bottomScroll < 60) this.scrollMessagesCount = 0
			this.scrollIcon = bottomScroll > 500 || this.scrollMessagesCount
		},
		loadMoreMessages() {
			if (this.loadingMessages) return

			setTimeout(
				() => {
					if (this.loadingMoreMessages) return

					if (this.messagesLoaded || !this.roomId) {
						this.loadingMoreMessages = false
						this.showLoader = false
						return
					}

					this.$emit('fetch-messages')
					this.loadingMoreMessages = true
				},
				// prevent scroll bouncing speed
				500
			)
		},
		messageActionHandler({ action, message }) {
			switch (action.name) {
				case 'replyMessage':
					this.initReplyMessage = message
					return
				case 'editMessage':
					this.initEditMessage = message
					return
				case 'deleteMessage':
					return this.$emit('delete-message', message)
				case 'selectMessages':
					this.selectedMessages = [message]
					this.messageSelectionEnabled = true
					return
				default:
					return this.$emit('message-action-handler', { action, message })
			}
		},
		messageSelectionActionHandler(action) {
			this.$emit('message-selection-action-handler', {
				action,
				messages: this.selectedMessages
			})
			this.resetMessageSelection()
		},
		sendMessageReaction(messageReaction) {
			this.$emit('send-message-reaction', messageReaction)
		},
		getBottomScroll(element) {
			const { scrollHeight, clientHeight, scrollTop } = element
			return scrollHeight - clientHeight - scrollTop
		},
		scrollToBottom() {
      this.$nextTick(() => {
        this.$refs.scrollContainer.scrollToBottom()
      })
		},
		openFile({ message, file }) {
			this.$emit('open-file', { message, file })
		},
		openUserTag(user) {
			this.$emit('open-user-tag', user)
		}
	}
}
</script>
