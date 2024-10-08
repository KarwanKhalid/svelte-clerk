<script lang="ts">
	import type {
		CheckAuthorizationWithCustomPermissions,
		OrganizationCustomPermissionKey,
		OrganizationCustomRoleKey
	} from '@clerk/types';
	import type { Snippet } from 'svelte';
	import { useClerkContext } from '$lib/context.js';

	type ProtectProps =
		| {
				condition?: never;
				role: OrganizationCustomRoleKey;
				permission?: never;
				children: Snippet;
				fallback?: Snippet;
		  }
		| {
				condition?: never;
				role?: never;
				permission: OrganizationCustomPermissionKey;
				children: Snippet;
				fallback?: Snippet;
		  }
		| {
				condition: (has: CheckAuthorizationWithCustomPermissions) => boolean;
				role?: never;
				permission?: never;
				children: Snippet;
				fallback?: Snippet;
		  }
		| {
				condition?: never;
				role?: never;
				permission?: never;
				children: Snippet;
				fallback?: Snippet;
		  };

	const { role, condition, permission, children, fallback }: ProtectProps = $props();

	const ctx = useClerkContext();

	const has = (params: Parameters<CheckAuthorizationWithCustomPermissions>[0]) => {
		if (!params?.permission && !params?.role)
			throw new Error(
				'Missing parameters. The prop permission or role is required to be passed. Example usage: `has({permission: "org:posts:edit"})`'
			);
		if (!ctx.auth.orgId || !ctx.auth.userId || !ctx.auth.orgRole || !ctx.auth.orgPermissions)
			return false;
		if (params.permission) return ctx.auth.orgPermissions.includes(params.permission);
		if (params.role) return ctx.auth.orgRole === params.role;
		return false;
	};

	const isAuthorized = $derived.by(() => {
		const { userId } = ctx.auth;

		if (!userId) return false;

		if (typeof condition === 'function') {
			return condition(has);
		}

		if (role || permission) {
			// @ts-expect-error: Fix types later
			return has({ role, permission });
		}

		return true;
	});
</script>

{#if isAuthorized}
	{@render children()}
{:else}
	{@render fallback?.()}
{/if}
