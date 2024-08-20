### 说明
本仓库是自学swift的过程, 在这个过程中遇到相似的概念时, 笔者会穿插很多其他语言相关的比较, 特别是C++和OC. 整个过程中有些地方会比较深入, 会涉及到汇编,操作系统,设计架构等. 

### 特有名词

|笔者定义|说明|
|:-|:-|
|`lb::`|`namespace lb  = std`|
|OS|操作系统|
|kernel|操作系统的内核|
|linux|Linux|
|macos|Mac OSX|
|darwin|Mac内核|
|thread|线程|
|proc|进程|
|tid|线程id|
|pid|进程id|
|ppid|父进程id|
|exec file|可执行文件(execuable file)|
|comp|编译|
|compr|编译器|
|comptime|编译时|
|runtime|运行时|
|CRT|c运行时库|
|value type|值类型|
|ref type|引用类型|
|call stack|调用栈, call stack|
|stack frame|栈桢|
|asm|`AT&T汇编`|
|arm|arm64`|
|intel|x86_64|
|obj|笔者将用户声明的所有标识符统称为对象|
|decl|声明declaration|
|defi|定义definiation|
|constant|常量|
|comp constant|编译期常量|
|ctor|swift或c++中的构造函数[^ann-init]|
|init|swift或c++中的初始化动作|
|assgin|赋值操作|
|lambda|匿名函数或匿名闭包, 笔者基本上会将闭包称为lambda|
|array|数组|
|map|字典|
|tuple|swift中的元组, c++中的tuple|
|block|OC中的lambda|

> 这些词语有的会经常使用, 出现了就表示文档中对应的概念


### 测试环境
1. 大部分在neovim中编写
2. xcode

# 环境配置
### neovim
版本`0.10.1`

直接使用homebrew下载

```
brew install neovim
```


# 包管理器
笔者使用的是[packer](https://github.com/wbthomason/packer.nvim), 请自行查阅文档


# 配置
### lsp相关的依赖
以下只截取部分的packer配置
```lua
-- LSP
use {
    "neovim/nvim-lspconfig",                        -- neovim自带了LSP的客户端接口, 这个是官方提供的客户端配置插件, 是必须的, 但真正不对它配置, 会利用 mason和mason-lspconfig来间接配置, 减少配置复杂性
    "williamboman/mason.nvim",                      -- mason来管理所有的LSP的配置(全是自动化的)
    "williamboman/mason-lspconfig.nvim",

    'hrsh7th/cmp-nvim-lsp',                         -- 片段源, 数据来源源头是LSP服务器器, 只不过该插件间接从neovim内置的LSP拿数据
    'hrsh7th/cmp-buffer',                           -- 片段源来自当前buffer(vim中的buffer即为磁盘文件在vim中的缓存)
    'hrsh7th/cmp-path',                             -- 片段源来自系统或用户文件路径
    'hrsh7th/cmp-cmdline',                          -- 片段源数据来源于解析命令
    'hrsh7th/nvim-cmp',                             -- 核心引擎(最后加载)
                                                    -- 这些插件只获取了最核心的片段获取数据, 不佬渲染UI
                                                    --
    {
        'L3MON4D3/LuaSnip',
        tag = "v2.*", 
        run = "make install_jsregexp"
    },
    'saadparwaiz1/cmp_luasnip',                     -- nvim-cmp官方要求要装的, 这2个是将上面片段源渲染成UI(如补全的弹框)
    'rafamadriz/friendly-snippets',			        -- 常用语言的代码片段(如xcode中键入if后, 会有一堆if补全出来)
}

-- 美化LSPUI
use({ 
    'nvimdev/lspsaga.nvim',
    after = 'nvim-lspconfig',
    config = tierry_saga
})

-- lsp代码高亮渲染
use {
    'nvim-treesitter/nvim-treesitter',              -- 语法高亮
    run = function()
        local ts_update = require('nvim-treesitter.install').update({ with_sync = true })
        ts_update()
    end,
}
```

<br/>

### lsp相关的配置
这里涉及到了CPP相关的配置(`clangd`), 与其相关的lsp服务器可以直接被[Mason](https://github.com/williamboman/mason.nvim)管理, 与swift不相关. 若只配置swift则不需要它(<font color = red>注释`__code_comment_herer`这一行</font>)
```lua
local ALL_SUFIX = {
    h   = true, 
    c   = true, 
    cc  = true,
    cpp = true, 
    hpp = true, 
    m   = true, 
    mm  = true, 
    swift = true,
    sh  = true
}

local FILE_SUFIX = string.lower(vim.api.nvim_buf_get_name(0)):match(".+%.(%w+)$")
if not ALL_SUFIX[FILE_SUFIX] then 
    return
end


local SUFIX_MAP_LSP = {
	cpp = 'clangd',
	c   = 'clangd',
	m   = 'clangd',
	mm  = 'clangd',
	hpp = 'clangd',
	h   = 'clangd',
	cc  = 'clangd',
	swift= 'sourcekit'
}

-- 如clangd, mason会自动找到Mac系统中Xcode自带的clangd
local ALL_LSP = {}
do 
	ALL_LSP.clangd = {
		surport = true, 
		opt = {
			filetypes = {"c", "cpp","objc", "objcpp", "hpp"}
		}
	}

	ALL_LSP["sourcekit"] = {
		-- mason不支持, 要自己配置
		surport = false,		
		opt = {
            capabilities = {
                workspace = {
                    didChangeWatchedFiles = {
                        dynamicRegistration = true,
                    },
                }
            },
			filetypes = {"swift", "c", "cpp", "objective-c", "objective-cpp"},
			cmd = {'sourceKit-lsp'},
		}
	}


	local LSP_NAME = {}
	do
	    for k, val in pairs(ALL_LSP) do
		if val.suport then
			table.insert(LSP_NAME, k)               -- __code_comment_herer
		end
	    end
	end

	require("mason").setup()
	require("mason-lspconfig").setup({
	     ensure_installed = LSP_NAME
	})

	-- custom lsp
end




local luasnip = require("luasnip")
local cmp = require'cmp'


-- 当只有一个条目时, 立即在TAB上确认(摁TAB后直接选中关闭框)
-- 这里有个问题: 当光标前有字符时, 若当前并未展示可选的视图, 摁下TAB键会卡顿一下, 弹出视图, 这个功能不需要, 所以下面TAB去掉了
local has_words_before = function()
  unpack = unpack or table.unpack
  local line, col = unpack(vim.api.nvim_win_get_cursor(0))
  return col ~= 0 and vim.api.nvim_buf_get_lines(0, line - 1, line, true)[1]:sub(col, col):match("%s") == nil
end


cmp.setup({
    experimental = {
        ghost_text = false,
    },

    -- 安装的什么这里写什么 L3MON4D3/luaSnip saadparwaiz1/cmp_luasnip
    snippet = {
      expand = function(args)
        --vim.fn["luasnip"](args.body)        
        require('luasnip').lsp_expand(args.bod)
      end,
    },

    -- 弹框带边框
    window = {
        completion = cmp.config.window.bordered(),
        documentation = cmp.config.window.bordered(),
    },

    -- 映射
    mapping = cmp.mapping.preset.insert({
        -- 出现补全(show)
        ['<M-s>'] = cmp.mapping(cmp.mapping.complete(), { 'i', 'c' }),

        -- Enter确认
        ['<CR>'] = cmp.mapping(function(fallback)
            if cmp.visible() then
                if luasnip.expandable() then
                    luasnip.expand()
                else
                    cmp.confirm({
                        select = true,
                    })
                end
            else
                fallback()
            end
        end),

        -- 下一个, 若只有1个选项时, 摁TAB后立即选中结束
        ['<Tab>'] = cmp.mapping(function(fallback)
            if cmp.visible() then
                if #cmp.get_entries() == 1 then
                    cmp.confirm({ select = true })
                else
                    cmp.select_next_item()
                end
            -- elseif has_words_before() then
            --     cmp.complete()
            --     if #cmp.get_entries() == 1 then
            --         cmp.confirm({ select = true })
            --     end
            elseif luasnip.locally_jumpable(1) then
                luasnip.jump(1)
            else
                fallback()
            end
        end, { "i", "s" }),

        -- 上一个
        ["M-h"] = cmp.mapping(function(fallback)
            if cmp.visible() then
                cmp.select_prev_item()
            elseif luasnip.locally_jumpable(-1) then
                luasnip.jump(-1)
            else
                fallback()
            end
        end, { "i", "s" }),

        ['<C-u>'] = cmp.mapping(cmp.mapping.scroll_docs(-4), { 'i', 'c' }),
        ['<C-d>'] = cmp.mapping(cmp.mapping.scroll_docs(4), { 'i', 'c' }),
    }),
    
    -- 补全源(注意顺序)
    sources = cmp.config.sources({
      { name = 'nvim_lsp' },
      { name = 'luasnip' }, 
    },
    {
      { 
          name = 'buffer' ,
          option = {
            get_bufnrs = function()
                return vim.api.nvim_list_bufs()
            end
          }
      },
      { name = 'treesitter' },
      { name = 'path'}
    }),

    -- 注释时不需要补全 
    enabled = function()
      local context = require 'cmp.config.context'
      if vim.api.nvim_get_mode().mode == 'c' then
        return true
      else
        return not context.in_treesitter_capture("comment") 
          and not context.in_syntax_group("Comment")
      end
    end
})


-- cmd补全
cmp.setup.cmdline({ '/', '?' }, { mapping = cmp.mapping.preset.cmdline(),
sources = { { name = 'buffer' } } })

cmp.setup.cmdline(':', { mapping = cmp.mapping.preset.cmdline(), sources =
cmp.config.sources({ { name = 'path' } }, { { name = 'cmdline' } }), matching =
{ disallow_symbol_nonprefix_matching = false } })



-- 代码中错误,警告等配置
vim.diagnostic.config({
    virtual_text = false,        -- 不要将错误主动提示在行后
    sgins = true,
    float = {
        border = "rounded",
        source = "always",
    },
})
local signs = { Error = "󰅙", Info = "󰋼", Hint = "󰌵", Warn = "" }
for type, icon in pairs(signs) do
    local hl = "DiagnosticSign" .. type
    vim.fn.sign_define(hl, { text = icon, texthl = hl, numhl = hl })
end


local CAPABILITIES = require('cmp_nvim_lsp').default_capabilities()
local LSP_CFG = require('lspconfig')
do
    for lsp, opt in pairs(ALL_LSP) do
	    local file_lsp = SUFIX_MAP_LSP[FILE_SUFIX]
	    if file_lsp == lsp then
            if not opt.capabilities then 
                opt.capabilities = CAPABILITIES 
            end
		    LSP_CFG[lsp].setup(opt.opt)
	    end
    end
end
```

<br/>

### sourceKit-lsp
`sourceKit-lsp`在Mac环境下自带, 没有的话去下载. 在配置中使用的名字是:`ALL_LSP["sourcekit"]`, 注意不要搞错了


# 初始化swift工程
### swift命令生成工程
必须先在命令行创建工程, 这样会生成配置文件, lsp才会起效

```bash
cd /tmp
mkdir test
swift package init --name hello-world --type executable


tree
 .
├──  .build
│  ├──  arm64-apple-macosx
│  │  └──  debug
│  │     ├──  index
│  │     │  ├──  db
│  │     │  │  └──  v13
│  │     │  │     └──  saved
│  │     │  │        ├──  data.mdb
│  │     │  │        └──  lock.mdb
│  │     │  └──  store
│  │     │     └──  v5
│  │     │        └──  units
│  │     └──  ModuleCache
│  │        ├──  1LAWH5RUC41NQ
│  │        │  ├──  _SwiftConcurrencyShims-39ZO3A1THQ3HL.pcm
│  │        │  ├──  _SwiftConcurrencyShims-39ZO3A1THQ3HL.pcm.timestamp
│  │        │  ├──  SwiftShims-39ZO3A1THQ3HL.pcm
│  │        │  └──  SwiftShims-39ZO3A1THQ3HL.pcm.timestamp
│  │        ├──  _Concurrency-22PCT9WK9OWDN.swiftmodule
│  │        ├──  _StringProcessing-3A1W7CALWUB5F.swiftmodule
│  │        ├──  modules.timestamp
│  │        └──  Swift-3QIFT6LHSLSXN.swiftmodule
│  ├──  artifacts
│  ├──  checkouts
│  ├──  repositories
│  └──  workspace-state.json
├──  .gitignore
├──  Package.swift
└──  Sources
   └──  main.swift
```

> 上述文档结构中文件类型所展示的图标在README上是看不见的, 需要相关字体, 这里不解释了, 最后看一下效果


[^ann-init]: swift中的构造方法是init, 但笔者这里将和c++中一样使用constructor来统一概念. 而init同统一成初始化
