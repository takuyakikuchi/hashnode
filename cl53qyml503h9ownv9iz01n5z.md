## The difference between CSR, SSR and SSG

# Difference between CSR, SSR, and SSG
## CSR
Client-side rendering.<br />
** A method of executing JavaScript on the browser to generate the DOM and display the content after it is mounted. **<br />
Initial loading of the page does not display any content, it will be displayed after hydration.<br />
React applications created with Create React App are rendered this way.


![no-pre-rendering.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656757383519/Dl4woz07P.png align="left")
(Image source: https://nextjs.org/learn/basics/data-fetching/pre-rendering)

## SSR
Server-side rendering.<br />
A method of evaluating and executing components on the server side and delivering the results in HTML and minimal JavaScript.<br />
**Each time a request is made to the server, the HTML is processed and generated on the server side. **<br />
Nuxt.js, Next.js, etc. are rendered this way.<br />
The content is displayed from the initial load, and then becomes interactive by Hydration. (e.g., `<Link />` makes it jumpable)<br />
It is considered better performance and SEO friendly than CSR.


![pre-rendering.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656757395936/-_V5fb7so.png align="left")
(Image source: https://nextjs.org/learn/basics/data-fetching/pre-rendering)


![server-side-rendering.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656757424170/xtZUPU9t7.png align="left")
(Image source: https://nextjs.org/learn/basics/data-fetching/two-forms)

## SSG
Server-side generator.<br />
Like SSR, HTML is generated on the server side first.<br />
The difference between SSG and SSR is that **HTML is generated at build time, and content is delivered from the CDN each time a request is made. **<br />
This is used for static pages such as blogs, help sites, and e-commerce product lists.
Better performance than SSR because HTML is generated at build time.


![static-generation.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656757438571/a3FzVuBrf.png align="left")
(Image source: https://nextjs.org/learn/basics/data-fetching/two-forms)

# Regarding the difference in use.

**"Is it ok to pre-render a page prior to the user's request?" **<br />
If yes, use SSG.<br />
If no, use SSR or CSR.<br />
Next.js can set SSG or SSR for each page.

[Click here for the Japanese article](https://zenn.dev/takuyakikuchi/articles/2f7e54bdafce52)

# References
- [Pre-rendering - Pre-rendering and Data Fetching | Learn Next.js](https://nextjs.org/learn/basics/data-fetching/pre-rendering)
- [Two Forms of Pre-rendering - Pre-rendering and Data Fetching | Learn Next.js](https://nextjs.org/learn/basics/data-fetching/two-forms)
- (Japanese article) [SSR、SSG、Client Side Renderingの違いをまとめた - Qiita](https://qiita.com/akashixi/items/84cd79e090a283bb8c67)