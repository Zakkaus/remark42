# xzOS theme branch

`xzos-theme` restyles the comment widget to match the xzOS blog: GNU Unifont at
16px, the blog's day palette for `theme: 'light'` and its night palette for
`theme: 'dark'`, square corners, no shadows. It is cut from upstream tag
`v1.18.0` and keeps every upstream name, so a rebase touches few lines.

## What the branch changes

| File | Change |
| --- | --- |
| `frontend/apps/remark42/app/styles/custom-properties.css` | Variable values only: day palette in `:root`, night palette in `:root .dark`, night code colours on `:root .dark .raw-content` |
| `frontend/apps/remark42/app/styles/global.css` | Imports `fonts.css`; appends one block: Unifont 16px everywhere, radius 0, a solid focus ring, a 3:1 edge on the comment field, placeholder colour, accent `strong` and secondary `em` |
| `frontend/apps/remark42/app/styles/fonts.css` (new) | `@font-face` rules for the Latin, CJK chunk and common-CJK subsets; the common file stays last |
| `frontend/apps/remark42/app/assets/fonts/` (new) | The woff2 subsets and `unifont-OFL.txt` (SIL OFL 1.1) |
| `frontend/apps/remark42/webpack.config.js` | Emits `.woff2` to `/web/fonts/` |
| `backend/app/rest/api/middleware.go` | CSP `font-src 'none'` becomes `font-src 'self'`, or the browser refuses the fonts |
| `.github/workflows/xzos-image.yml` (new) | Builds and pushes the image on `xzos-*` tags |

Three component styles are edited, because a variable value cannot fix them:

- `components/button/button.module.css` and `components/auth/components/button.module.css`:
  text on a filled accent button uses the page background colour. The night
  accent is light, so upstream's white text would be unreadable on it, and the
  same variables also colour light text elsewhere.
- `components/auth/auth.module.css`: the sign-in dropdown draws a 1px border
  instead of its box shadow. The checked provider's text uses the darker accent,
  because the accent on its 10% accent tint stays under 4.5:1 by day.

## Rebase onto a new upstream tag

```sh
git -C <repo> fetch upstream --tags
git -C <repo> rebase --onto vX.Y.Z v1.18.0 xzos-theme   # replace v1.18.0 with the current base tag
```

Conflicts land in `custom-properties.css`: keep upstream's new names and order,
and give each new or re-used `--colorN` the day or night value of its role.
`grep -rn 'var(--color' frontend/apps/remark42/app/components` shows where a
variable is used; a variable used under `.dark`, `.themeDark` or
`:global(.dark)` also needs a value in the `:root .dark` block. Then update the
base tag in this file and check:

```sh
cd frontend/apps/remark42 && pnpm i && pnpm lint type-check translation-check test && pnpm build
cd backend && go test ./app/rest/...
```

Look at the widget in both themes before tagging.

## Build and deploy

Push a tag named `xzos-<upstream version>-<n>`, for example `xzos-v1.18.0-1`.
The `xzos-image` workflow builds the repository `Dockerfile` (backend tests
skipped, frontend lint and tests run) and pushes
`ghcr.io/zakkaus/remark42:<tag>` for `linux/amd64`. It can also be started by
hand from the Actions tab with an optional image tag.

Deploy by replacing `umputun/remark42:<version>` with
`ghcr.io/zakkaus/remark42:<tag>`. The environment and the `/srv/var` volume are
unchanged. A new ghcr package is private: make it public in the package
settings or log the server in to ghcr.io before pulling.

The blog chooses the palette through the embed's `theme` parameter
(`'light'` for day, `'dark'` for night) and switches it with
`window.REMARK42.changeTheme()`.
