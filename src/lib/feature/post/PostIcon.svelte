<script lang="ts">
  import type { Community, Person, SubscribedType } from '$lib/api/types'
  import { settings, type View } from '$lib/app/settings.svelte'
  import { userLink } from '$lib/app/util.svelte'
  import Avatar from '$lib/ui/generic/Avatar.svelte'
  import { Material, Popover } from 'mono-svelte'

  interface Props {
    community?: Community
    subscribed?: SubscribedType
    user?: Person
    view?: View
  }

  let { community, subscribed, user, view = 'cozy' }: Props = $props()

  let size = $derived(view === 'compact' ? 24 : 32)
  let overlaySize = $derived(view === 'compact' ? 14 : 18)
</script>

{#snippet communityPopover()}
  {#if community && subscribed}
    <Material
      color="uniform"
      rounding="2xl"
      elevation="high"
      class="max-w-2xl w-screen max-h-128 overflow-auto"
      data-autoclose="false"
    >
      {#await import('../community/CommunityHeader.svelte') then { default: CommunityHeader }}
        <CommunityHeader {community} {subscribed} />
      {/await}
    </Material>
  {/if}
{/snippet}

{#snippet communityAvatar()}
  {#if community?.nsfw && settings.nsfwBlur}
    <div
      style="width: {size}px; height: {size}px;"
      class="bg-red-400 rounded-xl"
    ></div>
  {:else}
    <Avatar
      url={community?.icon}
      width={size}
      alt={community?.name}
      circle={false}
      class="group-hover/btn:scale-90 group-active/btn:scale-[.85] transition-transform"
    />
  {/if}
{/snippet}

{#if settings.posts.postIcon === 'community'}
  <Popover>
    {#snippet target(attachment)}
      <button
        {@attach attachment}
        class={[
          'row-span-2 shrink-0 mr-2 self-center group/btn',
          'bg-slate-200 dark:bg-zinc-800 rounded-lg cursor-pointer',
        ]}
      >
        {@render communityAvatar()}
      </button>
    {/snippet}
    {#snippet popover(open)}
      {#if open}
        {@render communityPopover()}
      {/if}
    {/snippet}
  </Popover>

{:else if settings.posts.postIcon === 'combined'}
  <Popover>
    {#snippet target(attachment)}
      <button
        {@attach attachment}
        class="row-span-2 shrink-0 mr-2 self-center relative group/btn cursor-pointer bg-slate-200 dark:bg-zinc-800 rounded-lg"
        style="width: {size}px; height: {size}px;"
      >
        {@render communityAvatar()}
        <Avatar
          url={user?.avatar}
          width={overlaySize}
          alt={user?.name}
          circle={true}
          class="absolute bottom-0 right-0 translate-x-1/4 translate-y-1/4 ring-2 ring-white dark:ring-zinc-900"
        />
      </button>
    {/snippet}
    {#snippet popover(open)}
      {#if open}
        {@render communityPopover()}
      {/if}
    {/snippet}
  </Popover>

{:else}
  <Popover>
    {#snippet target(attachment)}
      <button
        {@attach attachment}
        class={[
          'row-span-2 shrink-0 mr-2 self-center group/btn',
          'bg-slate-200 dark:bg-zinc-800 rounded-lg cursor-pointer',
        ]}
      >
        <Avatar
          url={user?.avatar}
          width={size}
          alt={user?.name}
          circle={true}
          class="group-hover/btn:scale-90 group-active/btn:scale-[.85] transition-transform"
        />
      </button>
    {/snippet}
    {#snippet popover(open)}
      {#if open && user}
        <Material
          color="uniform"
          rounding="2xl"
          elevation="high"
          class="max-w-sm w-screen max-h-128 overflow-auto"
          data-autoclose="false"
        >
          {#await import('$lib/ui/generic/EntityHeader.svelte') then { default: EntityHeader }}
            <EntityHeader
              avatarCircle
              avatar={user.avatar}
              name={user.display_name ?? user.name}
              banner={user.banner}
              bio={user.bio}
              url={userLink(user)}
              stats={[]}
            />
          {/await}
        </Material>
      {/if}
    {/snippet}
  </Popover>
{/if}
