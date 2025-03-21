<script lang="ts">
	import auth from '$lib/utils/authService';
	import {
		user,
		auth0Client,
		isAuthenticated,
		conversationsList,
		currentSlug
	} from '$lib/stores/store';
	import { onMount } from 'svelte';
	import type { ConversationType } from '$lib/types/types';
	import { goto } from '$app/navigation';
	import { db } from '$lib/db/db';
	import { liveQuery } from 'dexie';

	let userPictureUrl: string;
	let userName: string;
	let isSidebarOpen = true;

	onMount(async () => {
		// Check localStorage for sidebar state
		const sidebarState = localStorage.getItem('Sidebar');
		isSidebarOpen = sidebarState !== 'closed';

		user.subscribe((value) => {
			if (value) {
				// @ts-expect-error
				userPictureUrl = value.picture;
				// @ts-expect-error
				userName = value.name;
			}
		});
		try {
			// Welcome Message
			const storeConversations = liveQuery(() => db.conversations.toArray());
			storeConversations.subscribe(async (value) => {
				console.log(value);
				// Check if a conversation with the id "welcome-message" exists
				const exists = value.some((conversation) => conversation.id === 'welcome-message');
				if (exists) {
					console.log('Welcome message conversation exists.');
					conversationsList.set(value);
				} else {
					console.log('Welcome message conversation does not exist.');
					const addConversation = await db.conversations.add({
						id: 'welcome-message',
						name: 'Welcome to Grade AI',
						slug: 'welcome-to-grade-ai',
						content: [
							{
								prompt: {
									content: 'What is Grade AI?',
									sender: 'User',
									time: '2023-03-30T10:00:00.000Z'
								},
								response: {
									content: `<h1 class="text-4xl"><b>Grade AI is the best AI Chat for students ever made.</b></h1><br>
																<h2 class="text-2xl">1. We're optimized for students.</h2>
																<p>We have optimized each of our models for students, with custom prompting and much more to not only provide the answer, but to teach it.</p><br>
																<h2 class="text-2xl">2. We have multiple models, not just one.</h2>
																<p>Want to use Claude for code? We got you. DeepSeek r1 for math? Of course. ChatGPT 4o for picture analysis? Why not.
																When new models come out, we make them available within hours of release. </p><br>
																<h2 class="text-2xl">3. We're cheap. (10rs/month)</h2>
																<p>We're way cheaper than the price of ChatGPT or Claude.</p><br>
																<h2 class="text-2xl">What are you waiting for?</h2>
																Reply here to get started, or head over to conversation 1 to start a new chat.`,
									sender: 'Gemini',
									time: '2023-03-30T10:00:00.000Z'
								}
							}
						]
					});
				}
			});
		} catch (error) {
			console.log(error);
		}
	});

	function toggleSidebar() {
		isSidebarOpen = !isSidebarOpen;
		localStorage.setItem('Sidebar', isSidebarOpen ? 'open' : 'closed');
	}

	function login() {
		auth.loginWithPopup($auth0Client, {});
	}
	async function deleteChat(conversation: ConversationType) {
		const conversationSlug = conversation.slug;
		if (conversationSlug != $currentSlug || conversationSlug != 'welcome-to-grade-ai') {
			const conversationIndex = $conversationsList.findIndex(
				(conversation: any) => conversation.slug == conversationSlug
			);
			conversationsList.update((conversations: any) => {
				conversations.splice(conversationIndex, 1);
				return conversations;
			});
			const deleteConversation = await db.conversations.delete(conversation.id);
		}
	}
</script>

<div class="sidebar-container">
	<button
		class="toggle-sidebar-btn"
		on:click={toggleSidebar}
		aria-label={isSidebarOpen ? 'Close Sidebar' : 'Open Sidebar'}
	>
		<svg
			class="transition-transform duration-300 {isSidebarOpen ? 'rotate-180' : ''}"
			width="24"
			height="24"
			viewBox="0 0 24 24"
			fill="none"
			xmlns="http://www.w3.org/2000/svg"
		>
			<path
				d="M15 19L8 12L15 5"
				stroke="currentColor"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
		</svg>
	</button>

	<div class="side-bar {isSidebarOpen ? 'translate-x-0' : '-translate-x-full'}" id="side-bar">
		<div class="content h-screen">
			<div class="title flex items-center p-2">
				<p class="text-2xl font-bold bg-gradient-to-r from-primary to-secondary bg-clip-text text-transparent">Grade AI</p>
			</div>
			<div class="conversations mt-2 w-full list-none flex-row flex-wrap gap-2 overflow-y-scroll p-2">
				{#each $conversationsList as conversation}
					<a
						class="conversation btn btn-ghost flex w-full items-center overflow-hidden rounded-lg border-r-4 p-2 transition-all duration-300 hover:bg-base-200 hover:shadow-md {conversation.slug === $currentSlug ? 'bg-base-200 border-primary' : 'border-transparent'}"
						on:click={() => {
							currentSlug.set(conversation.slug);
						}}
						href="/{conversation.name.toLowerCase().replaceAll(' ', '-')}"
					>
						<svg
							width="20px"
							height="20px"
							viewBox="0 0 24 24"
							fill="none"
							xmlns="http://www.w3.org/2000/svg"
							class="transition-transform duration-300 group-hover:scale-110"
						>
							<path
								d="M17 3.33782C15.5291 2.48697 13.8214 2 12 2C6.47715 2 2 6.47715 2 12C2 13.5997 2.37562 15.1116 3.04346 16.4525C3.22094 16.8088 3.28001 17.2161 3.17712 17.6006L2.58151 19.8267C2.32295 20.793 3.20701 21.677 4.17335 21.4185L6.39939 20.8229C6.78393 20.72 7.19121 20.7791 7.54753 20.9565C8.88837 21.6244 10.4003 22 12 22C17.5228 22 22 17.5228 22 12C22 10.1786 21.513 8.47087 20.6622 7"
								stroke="currentColor"
								stroke-width="1.5"
								stroke-linecap="round"
							/>
							<path
								d="M8 12H8.009M11.991 12H12M15.991 12H16"
								stroke="currentColor"
								stroke-width="2"
								stroke-linecap="round"
								stroke-linejoin="round"
							/>
						</svg>
						<p class="flex-grow ml-2">{conversation.name}</p>
						<button
							aria-label="delete conversation"
							class="tooltip btn btn-ghost btn-sm opacity-0 group-hover:opacity-100 transition-opacity duration-300 hover:bg-error hover:text-error-content rounded-full p-1"
							data-tip="Delete Conversation"
							on:click={() => {
								deleteChat(conversation);
							}}
						>
							<svg
								width="16px"
								height="16px"
								viewBox="0 0 24 24"
								fill="none"
								xmlns="http://www.w3.org/2000/svg"
							>
								<path
									d="M4 6H20M16 6L15.7294 5.18807C15.4671 4.40125 15.3359 4.00784 15.0927 3.71698C14.8779 3.46013 14.6021 3.26132 14.2905 3.13878C13.9376 3 13.523 3 12.6936 3H11.3064C10.477 3 10.0624 3 9.70951 3.13878C9.39792 3.26132 9.12208 3.46013 8.90729 3.71698C8.66405 4.00784 8.53292 4.40125 8.27064 5.18807L8 6M18 6V16.2C18 17.8802 18 18.7202 17.673 19.362C17.3854 19.9265 16.9265 20.3854 16.362 20.673C15.7202 21 14.8802 21 13.2 21H10.8C9.11984 21 8.27976 21 7.63803 20.673C7.07354 20.3854 6.6146 19.9265 6.32698 19.362C6 18.7202 6 17.8802 6 16.2V6M14 10V17M10 10V17"
									stroke="currentColor"
									stroke-width="2"
									stroke-linecap="round"
									stroke-linejoin="round"
								/>
							</svg>
						</button>
					</a>
				{/each}
			</div>
			{#if $isAuthenticated}
				<div class="user-profile w-full p-4">
					<li class="list-none">
						<a href="/setting/user" title="Admin Dashboard" class="flex items-center group">
							<div class="relative">
								<img
									src={userPictureUrl}
									alt="profile"
									title={userName}
									class="w-10 h-10 cursor-pointer rounded-full ring-2 ring-primary ring-offset-2 ring-offset-base-300 transition-all duration-300 group-hover:ring-secondary group-hover:scale-105"
								/>
							</div>
							<p class="ml-3 text-sm font-medium transition-colors duration-300 group-hover:text-primary">{userName}</p>
						</a>
					</li>
				</div>
			{:else}
				<div class="login-button p-4">
					<button class="btn btn-primary w-full transition-all duration-300 hover:scale-105 hover:shadow-lg" on:click={login}>Login</button>
				</div>
			{/if}
		</div>
	</div>
</div>

<style>
	.sidebar-container {
		position: relative;
		height: 100vh;
	}

	.toggle-sidebar-btn {
		position: fixed;
		left: 0;
		top: 50%;
		transform: translateY(-50%);
		z-index: 100;
		background-color: var(--base-300);
		border: 1px solid var(--base-content);
		border-left: none;
		padding: 0.5rem;
		border-radius: 0 0.5rem 0.5rem 0;
		cursor: pointer;
		transition: all 0.3s ease-in-out;
		width: 40px;
		height: 80px;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.toggle-sidebar-btn:hover {
		background-color: var(--base-200);
		transform: translateY(-50%) scale(1.05);
	}

	.side-bar {
		position: fixed;
		left: 0;
		top: 0;
		min-width: 180px;
		width: 18vw;
		max-width: 17rem;
		height: 100vh;
		background-color: var(--base-300);
		border-right: 1px solid var(--base-content);
		backdrop-filter: blur(15px);
		transition: transform 0.3s ease-in-out;
		z-index: 99;
	}

	.conversations {
		height: calc(100vh - 100px);
		width: 100%;
		scrollbar-width: thin;
		scrollbar-color: var(--base-content) transparent;
	}

	.conversations::-webkit-scrollbar {
		width: 6px;
	}

	.conversations::-webkit-scrollbar-track {
		background: transparent;
	}

	.conversations::-webkit-scrollbar-thumb {
		background-color: var(--base-content);
		border-radius: 3px;
	}

	.login-button {
		position: absolute;
		bottom: 0;
		width: calc(100% - 32px);
	}

	.conversation {
		height: auto;
		min-height: 40px;
		max-width: 100%;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		position: relative;
	}

	.conversation p {
		font-size: 14px;
		text-align: left;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		margin: 0;
		color: var(--base-content);
	}

	.conversation:hover p {
		color: var(--primary);
	}

	/* Add smooth transitions for all interactive elements */
	.btn, .btn-ghost {
		transition: all 0.3s ease-in-out;
	}

	/* Add a subtle hover effect for the entire sidebar */
	.side-bar:hover {
		box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
	}
</style>
