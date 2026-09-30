# Publishing notes (for the maintainer)

1. Push this repo to https://github.com/ericgonzalez/Claude-Callprep (public).
2. Add GitHub topics: claude-code-plugin, claude-cowork, claude-skills, sales, call-prep, battle-card.
3. Test: Customize > Plugins > Add marketplace > ericgonzalez/Claude-Callprep > Sync > install,
   then ask for a call prep on a real company and confirm a PDF comes back.
4. Optional local check with Claude Code: `claude plugin validate .`
5. Submit at https://claude.ai/directory/manage : Submit new > Plugin bundle > repo URL > Validate >
   fix anything marked Blocks > Submit. A paid Claude plan is required. A person reviews new listings.
6. Update: raise `version` in .claude-plugin/plugin.json, commit, push to main. No resubmission needed.

The plugin name (call-prep-route) becomes permanent once listed. Change it in plugin.json and
marketplace.json before the first submission if you want something more distinctive.
