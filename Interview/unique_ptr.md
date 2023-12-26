#### unique_ptr 实现
``` CPP
class Deleter
{
public:
    Deleter() = default;

    template <typename Tp>
    void operator()(Tp *ptr)
    {
        if (ptr)
        {
            delete ptr;
        }
    }
};

template <typename Tp, typename Del = Deleter>
class UniquePtr
{
public:
    explicit UniquePtr() : ptr_(nullptr) {}

    explicit UniquePtr(Tp *ptr) : ptr_(ptr) {}

    explicit UniquePtr(UniquePtr &&unique_ptr)
    {
        ptr_ = std::forward<Tp *>(unique_ptr.ptr);
        deleter_ = std::forward<Del>(unique_ptr.deleter_);
        unique_ptr.ptr = nullptr;
    }

    ~UniquePtr()
    {
        deleter_(ptr_);
        ptr_ = nullptr;
    }

    UniquePtr(const UniquePtr &) = delete;
    UniquePtr operator=(const UniquePtr &) = delete;
    UniquePtr operator=(Tp *) = delete;

    void operator=(UniquePtr &&unique_ptr)
    {
        ptr_ = std::forward<Tp *>(unique_ptr.ptr);
        deleter_ = std::forward<Del>(unique_ptr.deleter_);
        unique_ptr.ptr = nullptr;
    }

    Tp &operator*() const noexcept
    {
        return *ptr_;
    }

    Tp *operator->() const noexcept
    {
        return ptr_;
    }

private:
    Tp *ptr_;
    Del deleter_;
};
```
