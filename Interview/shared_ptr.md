### shared_ptr 实现
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
class SharedPtr
{
public:
    SharedPtr(Tp *ptr = nullptr) : ptr_(ptr)
    {
        if (ptr_)
        {
            count = new size_t(1);
        }
        else
        {
            count = new size_t(0);
        }
    }

    ~SharedPtr()
    {
        (*count)--;
        if ((*count) == 0)
        {
            deleter_(ptr_);
            ptr_ = nullptr;
            delete count;
            count = nullptr;
        }
    }

    SharedPtr(const SharedPtr &ptr)
    {
        if (this != &ptr)
        {
            ptr_ = ptr;
            count = ptr->count;
            ++(*count);
        }
    }

    SharedPtr &operator=(const SharedPtr &ptr)
    {
        if (this != &ptr)
        {
            if (ptr_)
            {
                --(*count);
                if ((*count) == 0)
                {
                    delete ptr_;
                    delete count;
                }
                ptr_ = ptr;
                count = ptr->count;
                ++(*count);
            }
        }
        return *this;
    }

    SharedPtr(SharedPtr &&shared_ptr) : ptr_(nullptr), count(nullptr)
    {
        shared_ptr.swap(*this);
    }

    SharedPtr &operator=(SharedPtr &&shared_ptr)
    {
        SharedPtr(shared_ptr).swap(*this);

        return *this;
    }

    void swap(SharedPtr &other)
    {
        std::swap(ptr_, other.ptr_);
        std::swap(count, other.count);
    }

    Tp &operator*() const
    {
        return *ptr_;
    }

    Tp *operator->() const
    {
        return ptr_;
    }

private:
    Tp *ptr_;
    Del deleter_;
    size_t *count;
};
```
