
local __DARKLUA_BUNDLE_MODULES

__DARKLUA_BUNDLE_MODULES = {
    cache = {},
    load = function(m)
        if not __DARKLUA_BUNDLE_MODULES.cache[m] then
            __DARKLUA_BUNDLE_MODULES.cache[m] = {
                c = __DARKLUA_BUNDLE_MODULES[m](),
            }
        end

        return __DARKLUA_BUNDLE_MODULES.cache[m].c
    end,
}

do
    function __DARKLUA_BUNDLE_MODULES.a()
        local function VIDE_ASSERT(msg)
            error(msg, 0)
        end

        return VIDE_ASSERT
    end
    function __DARKLUA_BUNDLE_MODULES.b()
        local function inline_test()
            return debug.info(1, 'n')
        end

        local is_O2 = inline_test() ~= 'inline_test'

        return {
            strict = not is_O2,
            batch = false,
        }
    end
    function __DARKLUA_BUNDLE_MODULES.c()
        local throw = __DARKLUA_BUNDLE_MODULES.load('a')
        local flags = __DARKLUA_BUNDLE_MODULES.load('b')
        local scopes = {n = 0}

        local function ycall(fn, arg)
            local thread = coroutine.create(xpcall)

            local function efn(err)
                return debug.traceback(err, 3)
            end

            local resume_ok, run_ok, result = coroutine.resume(thread, fn, efn, arg)

            assert(resume_ok)

            if coroutine.status(thread) ~= 'dead' then
                return false, debug.traceback(thread, 'attempt to yield in reactive scope')
            end

            return run_ok, result
        end
        local function get_scope()
            return scopes[scopes.n]
        end
        local function assert_stable_scope()
            local scope = get_scope()

            if not scope then
                local caller_name = debug.info(2, 'n')

                return throw(string.format('cannot use %s() outside a stable or reactive scope', tostring(caller_name)))
            elseif scope.effect then
                throw(
[[cannot create a new reactive scope inside another reactive scope]])
            end

            return scope
        end
        local function push_child(parent, child)
            table.insert(parent, child)
            table.insert(child.parents, parent)
        end
        local function push_scope(node)
            local n = scopes.n + 1

            scopes.n = n
            scopes[n] = node
        end
        local function pop_scope()
            local n = scopes.n

            scopes.n = n - 1
            scopes[n] = nil
        end
        local function push_cleanup(node, cleanup)
            if node.cleanups then
                table.insert(node.cleanups, cleanup)
            else
                node.cleanups = {cleanup}
            end
        end
        local function flush_cleanups(node)
            if node.cleanups then... (226 KB restante(s))
