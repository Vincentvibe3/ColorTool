<script lang="ts">
    import { Navbar, Scaffold, SiteLayout, Text } from 'nota-ui';
    import type { LayoutData } from './$types';
    
    export let data: LayoutData;
</script>
<SiteLayout>
    <Navbar slot="navbar">
        <Text slot="title" tag="h1" heading6>Color Tool</Text>
    </Navbar>
    <Scaffold noPadding>
        <slot></slot>
    </Scaffold>
</SiteLayout>