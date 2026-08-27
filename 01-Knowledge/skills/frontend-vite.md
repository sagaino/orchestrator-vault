---
title: "Frontend Vite React TypeScript Enterprise Skill"
type: pattern
tags: ["skill", "frontend", "react", "vite", "tailwind", "shadcn", "react-query", "zod"]
created: 2026-08-27
updated: 2026-08-27
sources: ["[[01-Knowledge/patterns/frontend/project-skeleton-template]]", "pam-ad-web", "gallery-fmfu", "identity-kit-dashboard-web"]
---

# Frontend Vite React TypeScript Enterprise Skill (`frontend-vite@1.0.0`)

Panduan teknis dan standar arsitektur implementasi antarmuka frontend skala enterprise berbasis **React 19 + Vite + TypeScript + Tailwind CSS v4 + Shadcn UI + TanStack React Query v5 + React Hook Form + Zod**.

---

## 1. Arsitektur Folder Modul (*Feature-First Pattern*)

Setiap modul / halaman bisnis baru (`<FeatureName>`) **wajib** mengikuti pembagian struktur modular berikut:

```text
src/
├── pages/<FeatureName>/
│   ├── index.tsx                         # Page View & Orchestrator (Layout, Header, List, Modals)
│   ├── components/                       # Sub-komponen modular terfokus
│   │   ├── TableList.tsx                 # Render tabel data / grid kartu
│   │   ├── FormModal.tsx                 # Modal dialog Create / Edit
│   │   ├── FilterBar.tsx                 # Input pencarian & dropdown filter
│   │   └── DetailDrawer.tsx              # Panel detail / view modal
│   ├── hooks/                            # Custom React Query hooks (Query & Mutation)
│   │   └── use<FeatureName>.ts
│   ├── types/                            # TypeScript interfaces & types lokal
│   │   └── index.ts
│   └── schema/ (atau scheme/)            # Schema validasi form berbasis Zod
│       └── index.ts
│
├── services/                             # API Client service layer
│   └── <featureName>.ts                  # API caller object (AuthServices, UserServices, dll)
│
├── config/ (atau lib/)
│   ├── axios.ts                          # Axios instance terpusat (Signature, Auth Token, 401 Redirect)
│   ├── signature.ts                      # HMAC / timestamp security signature generator
│   └── constant/
│       ├── endpoint.ts                   # Konstanta ENDPOINTS URL backend
│       └── localstorage.ts               # Konstanta LOCALSTORAGE_KEY
│
├── components/                           # Komponen UI global (Layout, Header, Sidebar)
│   └── ui/                               # Vendor output Shadcn UI (JANGAN DIEDIT MANUAL)
│
└── routes/                               # Route definitions (React Router DOM v7)
```

---

## 2. Standar Service Layer & Endpoint (`src/services/` & `src/config/`)

### A. Definisi Endpoint (`src/config/constant/endpoint.ts`)
Semua path API backend **wajib didefinisikan sebagai konstanta**:
```typescript
export const ENDPOINTS = {
  LOGIN: "/api/v1/auth/login",
  USERS: "/api/v1/users",
  USER_DETAIL: (id: string | number) => `/api/v1/users/${id}`,
  PRODUCTS: "/api/v1/products",
  PRODUCT_DETAIL: (id: string | number) => `/api/v1/products/${id}`,
} as const;
```

### B. Service Object (`src/services/<featureName>.ts`)
Gunakan instance `axios` terpusat dan definisikan tipe kembalian Promise secara ketat:
```typescript
import { axios } from "@/config/axios";
import { ENDPOINTS } from "@/config/constant/endpoint";
import type { 
  ProductListResponse, 
  ProductDetailResponse, 
  CreateProductPayload, 
  UpdateProductPayload 
} from "@/pages/Product/types";

export const ProductServices = {
  getList: async (params?: { page?: number; limit?: number; search?: string }): Promise<ProductListResponse> => {
    const response = await axios.get<ProductListResponse>(ENDPOINTS.PRODUCTS, { params });
    return response.data;
  },

  getDetail: async (id: string): Promise<ProductDetailResponse> => {
    const response = await axios.get<ProductDetailResponse>(ENDPOINTS.PRODUCT_DETAIL(id));
    return response.data;
  },

  create: async (data: CreateProductPayload): Promise<ProductDetailResponse> => {
    const response = await axios.post<ProductDetailResponse>(ENDPOINTS.PRODUCTS, data);
    return response.data;
  },

  update: async (id: string, data: UpdateProductPayload): Promise<ProductDetailResponse> => {
    const response = await axios.put<ProductDetailResponse>(ENDPOINTS.PRODUCT_DETAIL(id), data);
    return response.data;
  },

  delete: async (id: string): Promise<void> => {
    await axios.delete(ENDPOINTS.PRODUCT_DETAIL(id));
  },
};
```

---

## 3. Standar State & Data Fetching (*TanStack React Query v5*)

Ditempatkan di dalam folder `src/pages/<FeatureName>/hooks/use<FeatureName>.ts`:
```typescript
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { ProductServices } from "@/services/product";
import type { CreateProductPayload, UpdateProductPayload } from "../types";
import { toast } from "sonner";

export const PRODUCT_QUERY_KEYS = {
  all: ["products"] as const,
  lists: () => [...PRODUCT_QUERY_KEYS.all, "list"] as const,
  list: (params?: Record<string, unknown>) => [...PRODUCT_QUERY_KEYS.lists(), params] as const,
  detail: (id: string) => [...PRODUCT_QUERY_KEYS.all, "detail", id] as const,
};

export const useProductQuery = (params?: { page?: number; limit?: number; search?: string }) => {
  return useQuery({
    queryKey: PRODUCT_QUERY_KEYS.list(params),
    queryFn: () => ProductServices.getList(params),
    staleTime: 5 * 60 * 1000, // 5 menit
  });
};

export const useProductDetailQuery = (id: string) => {
  return useQuery({
    queryKey: PRODUCT_QUERY_KEYS.detail(id),
    queryFn: () => ProductServices.getDetail(id),
    enabled: Boolean(id),
  });
};

export const useCreateProductMutation = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: CreateProductPayload) => ProductServices.create(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: PRODUCT_QUERY_KEYS.lists() });
      toast.success("Produk berhasil ditambahkan!");
    },
    onError: (error: any) => {
      toast.error(error.response?.data?.message || "Gagal menambahkan produk.");
    },
  });
};

export const useDeleteProductMutation = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (id: string) => ProductServices.delete(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: PRODUCT_QUERY_KEYS.lists() });
      toast.success("Produk berhasil dihapus!");
    },
  });
};
```

---

## 4. Standar Validasi Form (*React Hook Form + Zod*)

### A. Schema Zod (`src/pages/<FeatureName>/schema/index.ts`)
```typescript
import { z } from "zod";

export const productFormSchema = z.object({
  name: z.string().min(1, "Nama produk wajib diisi").max(100, "Maksimal 100 karakter"),
  description: z.string().optional(),
  price: z.coerce.number().min(1, "Harga minimal 1"),
  stock: z.coerce.number().min(0, "Stok tidak boleh negatif"),
  category: z.string().min(1, "Kategori wajib dipilih"),
});

export type ProductFormValues = z.infer<typeof productFormSchema>;
```

### B. Form Component (`src/pages/<FeatureName>/components/FormModal.tsx`)
```tsx
import React from "react";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { productFormSchema, type ProductFormValues } from "../schema";
import { useCreateProductMutation } from "../hooks/useProduct";
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import { Loader2 } from "lucide-react";

interface FormModalProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
}

export const FormModal: React.FC<FormModalProps> = ({ open, onOpenChange }) => {
  const createMutation = useCreateProductMutation();
  const {
    register,
    handleSubmit,
    reset,
    formState: { errors, isSubmitting },
  } = useForm<ProductFormValues>({
    resolver: zodResolver(productFormSchema),
    defaultValues: {
      name: "",
      description: "",
      price: 0,
      stock: 0,
      category: "",
    },
  });

  const onSubmit = async (values: ProductFormValues) => {
    await createMutation.mutateAsync(values);
    reset();
    onOpenChange(false);
  };

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="sm:max-w-lg bg-slate-900 border-slate-800 text-slate-100">
        <DialogHeader>
          <DialogTitle className="text-lg font-semibold text-slate-100">Tambah Produk Baru</DialogTitle>
        </DialogHeader>

        <form onSubmit={handleSubmit(onSubmit)} className="space-y-4 mt-2">
          <div className="space-y-1.5">
            <Label htmlFor="name" className="text-sm text-slate-200">Nama Produk</Label>
            <Input
              id="name"
              {...register("name")}
              placeholder="Contoh: Kamera Keamanan CCTV"
              className="bg-slate-950 border-slate-800 text-slate-100 focus:border-indigo-500"
            />
            {errors.name && <p className="text-xs text-rose-400">{errors.name.message}</p>}
          </div>

          <div className="grid grid-cols-2 gap-3">
            <div className="space-y-1.5">
              <Label htmlFor="price" className="text-sm text-slate-200">Harga (Rp)</Label>
              <Input
                id="price"
                type="number"
                {...register("price")}
                className="bg-slate-950 border-slate-800 text-slate-100 focus:border-indigo-500"
              />
              {errors.price && <p className="text-xs text-rose-400">{errors.price.message}</p>}
            </div>
            <div className="space-y-1.5">
              <Label htmlFor="stock" className="text-sm text-slate-200">Stok</Label>
              <Input
                id="stock"
                type="number"
                {...register("stock")}
                className="bg-slate-950 border-slate-800 text-slate-100 focus:border-indigo-500"
              />
              {errors.stock && <p className="text-xs text-rose-400">{errors.stock.message}</p>}
            </div>
          </div>

          <div className="space-y-1.5">
            <Label htmlFor="description" className="text-sm text-slate-200">Deskripsi</Label>
            <Textarea
              id="description"
              {...register("description")}
              rows={3}
              className="bg-slate-950 border-slate-800 text-slate-100 focus:border-indigo-500"
            />
          </div>

          <div className="flex justify-end gap-2 pt-3">
            <Button
              type="button"
              variant="ghost"
              onClick={() => onOpenChange(false)}
              disabled={isSubmitting || createMutation.isPending}
              className="text-slate-400 hover:text-slate-200 hover:bg-slate-800"
            >
              Batal
            </Button>
            <Button
              type="submit"
              disabled={isSubmitting || createMutation.isPending}
              className="bg-indigo-600 hover:bg-indigo-500 text-white font-medium"
            >
              {createMutation.isPending ? (
                <>
                  <Loader2 className="w-4 h-4 mr-2 animate-spin" />
                  Menyimpan...
                </>
              ) : (
                "Simpan Produk"
              )}
            </Button>
          </div>
        </form>
      </DialogContent>
    </Dialog>
  );
};
```

---

## 5. Standar Konfirmasi Aksi Destruktif (*SweetAlert2 / AlertDialog*)

Gunakan SweetAlert2 untuk konfirmasi penghapusan:
```typescript
import Swal from "sweetalert2";

export const confirmDelete = async (itemName: string): Promise<boolean> => {
  const result = await Swal.fire({
    title: "Konfirmasi Hapus",
    text: `Apakah Anda yakin ingin menghapus "${itemName}"?`,
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#ef4444",
    cancelButtonColor: "#334155",
    confirmButtonText: "Ya, Hapus",
    cancelButtonText: "Batal",
    background: "#0f172a",
    color: "#f8fafc",
    customClass: {
      popup: "border border-slate-800 rounded-xl",
    },
  });

  return result.isConfirmed;
};
```

---

## 6. Standar Halaman Container (`src/pages/<FeatureName>/index.tsx`)

Halaman container mengatur koordinasi komponen, filter state, dan modal:
```tsx
import React, { useState } from "react";
import { useProductQuery, useDeleteProductMutation } from "./hooks/useProduct";
import { TableList } from "./components/TableList";
import { FormModal } from "./components/FormModal";
import { FilterBar } from "./components/FilterBar";
import { confirmDelete } from "@/config/swal";
import { Button } from "@/components/ui/button";
import { Plus, Package } from "lucide-react";

export default function ProductPage() {
  const [search, setSearch] = useState("");
  const [isCreateOpen, setIsCreateOpen] = useState(false);
  const { data, isLoading, error } = useProductQuery({ search });
  const deleteMutation = useDeleteProductMutation();

  const handleDelete = async (id: string, name: string) => {
    const confirmed = await confirmDelete(name);
    if (confirmed) {
      await deleteMutation.mutateAsync(id);
    }
  };

  return (
    <div className="space-y-6 p-6">
      {/* Header Bar */}
      <div className="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
        <div className="flex items-center gap-3">
          <div className="p-2.5 rounded-xl bg-indigo-500/10 text-indigo-400 border border-indigo-500/20">
            <Package className="w-6 h-6" />
          </div>
          <div>
            <h1 className="text-2xl font-bold text-slate-100">Manajemen Produk</h1>
            <p className="text-sm text-slate-400">Kelola inventaris dan konfigurasi produk.</p>
          </div>
        </div>

        <Button
          onClick={() => setIsCreateOpen(true)}
          className="bg-indigo-600 hover:bg-indigo-500 text-white shadow-lg shadow-indigo-600/20 font-medium"
        >
          <Plus className="w-4 h-4 mr-2" />
          Tambah Produk
        </Button>
      </div>

      {/* Filter Bar */}
      <FilterBar search={search} onSearchChange={setSearch} />

      {/* Data Table */}
      <TableList
        items={data?.data ?? []}
        isLoading={isLoading}
        error={error}
        onDelete={handleDelete}
      />

      {/* Form Modal */}
      <FormModal open={isCreateOpen} onOpenChange={setIsCreateOpen} />
    </div>
  );
}
```

---

## 7. Standar Verifikasi & Build

* **Type Safety (`tsc -b`)**: Wajib zero compiler errors, no missing props, no unresolved types.
* **Linting (`npm run lint`)**: Sesuai aturan ESLint React Hooks.
* **Production Build (`npm run build`)**: Vite build wajib lulus tanpa bundle breaking errors.
